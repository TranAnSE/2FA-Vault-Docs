---
order: 73
---

# Auto-backup

Auto-backup runs scheduled, encrypted backups of your vault and pushes the resulting `.vault` file to one or more destinations you configure. It builds on the [manual encrypted backup](/usage/teams-and-backups/#encrypted-backups) flow.

## Prerequisites

Auto-backup is driven by the Laravel **queue worker**. A worker must be running for backup jobs to be dispatched:

```bash !#
php artisan queue:work
```

If you run 2FA-Vault in Docker, ensure the queue worker service (or container) is enabled. Without a running worker, scheduled backups will pile up in the queue and never execute.

## Enabling auto-backup

1. Go to **Settings → Backup → Auto-Backup**.
2. Enable **Auto-backup**.
3. Choose a **frequency**: _daily_, _weekly_, or _monthly_.
4. Choose a **time** in UTC (`HH:MM`). The scheduler checks every minute and runs the job when the configured time matches.
5. Add at least one [destination](#destinations).

## Destinations

You can configure up to **5 destinations** per account. Each backup run pushes the same encrypted `.vault` payload to every active destination. If a destination fails, the others still receive the backup — failures are isolated per destination.

### Local

Stores the backup file on the server filesystem.

| Field | Description |
|-------|-------------|
| Path | Absolute or relative path on the server. Must be writable by the web user. |

!!!warning Path safety
The local destination validates that the path stays within allowed directories to prevent path traversal. Choose a dedicated backup folder outside the public web root.
!!!

### S3-compatible

Works with AWS S3 and any S3-compatible provider: MinIO, Backblaze B2, Wasabi, DigitalOcean Spaces, Cloudflare R2, and others.

| Field | Description |
|-------|-------------|
| Endpoint | Endpoint URL. Leave empty for AWS S3; set it for MinIO, B2, Wasabi, etc. |
| Bucket | Bucket name |
| Region | Region (defaults to `us-east-1`) |
| Access key | Access key ID |
| Secret key | Secret access key (encrypted at rest, never returned by the API) |
| Path prefix | Optional prefix/folder for backup files |

### WebDAV

Stores the backup on a WebDAV server (Nextcloud, ownCloud, Synology, etc.).

| Field | Description |
|-------|-------------|
| URL | WebDAV base URL, e.g. `https://cloud.example.com/remote.php/dav/files/user/` |
| Username | WebDAV account username |
| Password | WebDAV account password (encrypted at rest, never returned by the API) |
| Path | Optional sub-folder |

!!!warning Nextcloud compatibility
`league/flysystem-webdav` may not fully support Nextcloud's non-standard WebDAV extensions. Use the standard `/remote.php/dav/files/<user>/` endpoint (not the legacy `/remote.php/webdav/` one) and test the connection after saving. If uploads fail, mount the backup folder via Nextcloud _External storage_ (a plain WebDAV/SMB share) and point 2FA-Vault at that share instead.
!!!

### Email

Sends the encrypted `.vault` file as an attachment to a configured address.

| Field | Description |
|-------|-------------|
| Email | Recipient address for the backup attachment |

!!!info Size limits
Email attachments are subject to your mail provider's size limits. For large vaults, prefer S3 or WebDAV over email.
!!!

## Testing a destination

After saving a destination, click **Test** to perform a zero-byte upload against it. The test verifies that the credentials and path are correct **without** storing a real backup file. Use it whenever you change credentials or endpoints.

## Backup format and notifications

Each run produces a single encrypted `.vault` file (the same format as a manual backup) named `backup-YYYY-MM-DD-HHMMSS.vault` and uploads it to every active destination.

When the run completes, a notification email is sent to your account email summarizing the result:

- Success: confirms the file name and the destinations that received it.
- Partial failure: lists each destination that failed, while noting that other destinations succeeded.

Failed destinations are also marked `failed` in their `last_run_status`, so you can spot problems at a glance in the destination list.

## Scheduling details

The scheduler runs every minute and dispatches a backup job for your account only when:

1. Auto-backup is enabled, **and**
2. The current UTC time matches your configured `HH:MM`, **and**
3. Enough time has elapsed since the last run according to your frequency (daily/weekly/monthly).

A `last_auto_backup_at` timestamp prevents double-dispatch within the same minute.

## Security

- Destination credentials (S3 secret keys, WebDAV passwords) are **encrypted at rest** and never returned in plain text by the API. GET responses expose only a masked `config_summary`.
- Credentials are never written to job logs, mail bodies, or API responses.
- The backup payload itself is already encrypted by `BackupService`, so it is safe to transmit over email or store on third-party object storage.
