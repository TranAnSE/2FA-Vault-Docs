---
order: 76
---

# Personal activity log

The personal activity log is a per-user audit trail of the actions performed on your account. It lets you review what happened and when, which is useful for spotting unexpected logins or changes.

## What is logged

The log records significant account and vault events. Each entry includes the action, the IP address and user agent of the client that performed it, optional metadata, and a timestamp.

Common logged actions include:

| Action | Description |
|--------|-------------|
| `login` | You signed in |
| `logout` | You signed out |
| `account_created` | A 2FA account was created |
| `account_updated` | A 2FA account was updated |
| `account_deleted` | A 2FA account was deleted |
| `preferences_changed` | A user preference was changed |
| `vault_locked` | The vault was locked |
| `vault_unlocked` | The vault was unlocked |
| `encryption_setup` | End-to-end encryption was enabled |
| `encryption_disabled` | End-to-end encryption was disabled |
| `backup_exported` | A backup was exported |
| `backup_imported` | A backup was imported |
| `note_created` | A secure note was created |
| `note_updated` | A secure note was updated |
| `note_deleted` | A secure note was deleted |
| `invitation_sent` | (Admin) An invitation email was sent |

## Viewing the log

Go to **Settings → Activity** to browse your activity log. Entries are listed newest first and paginated. Each row shows the action, the originating IP address, the client user agent, and the timestamp.

This log is **personal**: you only see actions performed on your own account. Administrators have a separate team/administration audit view and cannot read another user's personal activity log.

## Clearing the log

Click **Clear activity log** to permanently delete every entry. This action cannot be undone. Clearing the log does not affect any account data, settings, or sessions — it only removes the audit entries.

!!!info Retention
The personal activity log is independent of the [session list](/usage/session-management/) and the admin team activity log. Clearing one does not clear the others.
!!!
