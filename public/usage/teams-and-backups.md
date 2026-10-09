# Teams and encrypted backups

2FA-Vault extends the upstream 2FA-Vault workflow with team collaboration and encrypted backup endpoints.

## Teams

Teams let users share selected 2FA accounts with a controlled member list. A team owner can:

- create and rename a team
- invite users by email
- accept or cancel invitations
- remove members
- update member roles
- share or unshare selected accounts

Supported invitation/member roles are `admin`, `member`, and `viewer` — plus per-team **custom roles** (see [Roles & permissions](#roles--permissions)). Shared account access levels are `read`, `write`, and `admin`.

Team routes live under `/api/v1/teams/*` and require an authenticated Personal Access Token. Team authorization is enforced server-side, so clients should expect `403 Forbidden` when the current user is not allowed to view, invite, manage members, or share accounts for a team.

## Roles & permissions

Since v1.4.0, team authorization is driven by a per-team **permission matrix**. Every team gets four system presets whose behavior exactly matches earlier releases, and the owner can define additional custom roles.

| Permission | Gates |
|---|---|
| `members.invite` | Inviting members and managing pending invitations |
| `members.remove` | Removing members (revokes their shared access) |
| `roles.assign` | Changing member roles — a non-owner assigner can only assign roles whose permissions do not exceed their own |
| `accounts.share` | Sharing your own accounts with the team |
| `accounts.unshare_others` | Removing shares created by other members |
| `team.edit` | Renaming the team |
| `activity.view` | Viewing the team activity log |

System presets:

- **owner** — everything. Ownership is keyed on `owner_id`, never on a role string: `team.delete`, `roles.manage` (creating/editing/deleting role definitions), `activity.export`, and ownership transfer are **owner-locked** and can never be granted through the matrix.
- **admin** — everything except role assignment (matches the previous owner-only behavior).
- **member** — share own accounts; removing others' shares requires `accounts.unshare_others` (admin/owner).
- **viewer** — read-only, plus the historical quirk that viewers may still share (and unshare) **their own** accounts.

Custom roles are created by the team owner (`POST /api/v1/teams/{id}/roles`, UI: team page → Roles & permissions): a slug unique within the team, a display name, and a non-empty subset of the catalog above. Unknown permission values are rejected — there is no wildcard. A custom role can be deleted only while no member holds it and no pending invitation targets it.

Two safety rules apply everywhere:

- **Reserved owner slug** — the slug `owner` is rejected at invitation and accept time; a member row forged to `role: owner` gains nothing beyond its role definition.
- **Fail closed** — if a role definition disappears while a member still references it (a racing deletion), that member resolves to an empty permission set rather than an error.

## Encrypted key sharing

When vault encryption is enabled, a team owner can share an account with individual members without the server ever seeing the plaintext secret.

**How it works:**

1. Each user generates an RSA-2048 key pair in the browser. The public key is registered via `POST /api/v1/user/public-key`. The private key is stored only in the browser's IndexedDB.
2. The team owner fetches each member's public key (`GET /api/v1/teams/{id}/members/{userId}/public-key`) and wraps the account secret with RSA-OAEP encryption.
3. The wrapped keys are submitted in a single request to `POST /api/v1/teams/{id}/share-encrypted`. Each member gets their own row in `shared_accounts` with their personal `wrapped_key`.
4. When a member loads shared accounts, the response includes their `wrapped_key`. They decrypt it locally with their private key to obtain the plaintext secret and generate OTPs without any server-side decryption.

Members without a registered public key cannot receive encrypted shares. Revoking a share deletes the `wrapped_key` row immediately.

## Activity log

Every significant team action is recorded in a structured log visible to owners and admins at `/teams/:id/activity`.

Logged actions include:

| Action | Description |
|--------|-------------|
| `team.created` | Team was created |
| `team.updated` | Team name was changed |
| `member.invited` | A user was invited |
| `member.joined` | A user joined via invite code or accepted invitation |
| `member.left` | A member left voluntarily |
| `member.removed` | A member was removed by an admin or owner |
| `member.role_changed` | A member's role was updated |
| `account.shared` | An account was shared with the team |
| `account.unshared` | A shared account was removed |
| `invitation.cancelled` | A pending invitation was cancelled |

The activity log is paginated (50 per page). Owners can export the full log as a JSON file via `GET /api/v1/teams/{id}/activity/export`.

## Encrypted backups

The current backup flow uses `.vault` files. Export is initiated through `POST /api/v1/backups/export` with a backup password and an optional `include_groups` flag.

Import uses `POST /api/v1/backups/import` with a `backup_file`, optional format selection, optional password, conflict handling, and group import controls.

Supported conflict strategies are:

- `skip`
- `replace`
- `rename`

The backup metadata endpoint, `POST /api/v1/backups/metadata`, reads preview information from an uploaded backup file without importing it. `GET /api/v1/backups/info` returns backup-related stats for the authenticated user.

Legacy singular routes under `/api/v1/backup/*` remain available for backward compatibility, but new clients should prefer `/api/v1/backups/*`.

## Snapshots & versioning

Beyond downloadable `.vault` files, 2FA-Vault keeps **server-side snapshots** of your vault — row-level copies of your accounts and groups as they were at a point in time. You can create a snapshot manually from **Settings → Backup → Snapshots**, and the app creates snapshots automatically before every destructive restore and on a schedule you configure in **Settings → Backup → Auto-Backup**.

Snapshots live on the server, are encrypted at rest with the application key, and never expose your secrets through the API — end-to-end-encrypted secrets remain ciphertext inside a snapshot.

### Restoring a snapshot

Restoring always starts with a **dry-run diff**: the server compares the snapshot with your current vault over stable columns and reports which accounts would be created, updated, or deleted, then issues a single-use confirmation token. Matching is primarily done by account id; when an id no longer exists, accounts are matched by service + account name and every such fallback match is disclosed as a warning. Volatile fields (last-used timestamps, HOTP counters) are ignored by the diff.

- **Merge** re-creates accounts missing from your vault and restores lost fields on existing ones. Nothing is deleted. A single failing account does not affect the others.
- **Replace** wipes your vault and restores exactly the snapshot state. Accounts created after the snapshot are deleted. Replace requires a second, explicit step in the UI where you type `RESTORE`, and always keeps an automatic **pre-restore snapshot** of your vault as it was just before the wipe (that safety snapshot cannot be evicted by the quota during the restore).

If your vault changes between the diff and the restore, the restore is refused with `restore_state_changed` — re-run the diff and confirm again. Confirmation tokens expire after 10 minutes and are single-use.

!!!note Multi-node deployments
Restore confirmation tokens live in the application cache. On multi-node setups the cache store must be shared between nodes (e.g. `redis` or `database`) — a dry-run answered by one node and the restore by another would otherwise never see the token.
!!!

### Warnings to watch for

- **Master password changed since the snapshot** (`salt_changed_since_snapshot`): secrets that were end-to-end encrypted under your old password may not decrypt after a restore.
- **Application key rotated** (`unreadable_reason: app_key_rotated`): the snapshot can no longer be read or restored.

### Quota

Snapshots are subject to a per-user retention quota (defaults: 10 snapshots, 100 MB total — configurable by the instance administrator). When the quota is exceeded, the oldest evictable snapshot is removed. Automatic snapshots (scheduled and pre-restore) are evicted first and can **never** evict a manual snapshot you created deliberately.
