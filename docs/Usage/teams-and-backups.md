---
order: 95
---
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

Supported invitation/member roles are `admin`, `member`, and `viewer`. Shared account access levels are `read`, `write`, and `admin`.

Team routes live under `/api/v1/teams/*` and require an authenticated Personal Access Token. Team authorization is enforced server-side, so clients should expect `403 Forbidden` when the current user is not allowed to view, invite, manage members, or share accounts for a team.

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
