---
order: 95
---
# Teams and encrypted backups

2FA-Vault extends the upstream 2FAuth workflow with team collaboration and encrypted backup endpoints.

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

## Encrypted backups

The current backup flow uses `.vault` files. Export is initiated through `POST /api/v1/backups/export` with a backup password and an optional `include_groups` flag.

Import uses `POST /api/v1/backups/import` with a `backup_file`, optional format selection, optional password, conflict handling, and group import controls.

Supported conflict strategies are:

- `skip`
- `replace`
- `rename`

The backup metadata endpoint, `POST /api/v1/backups/metadata`, reads preview information from an uploaded backup file without importing it. `GET /api/v1/backups/info` returns backup-related stats for the authenticated user.

Legacy singular routes under `/api/v1/backup/*` remain available for backward compatibility, but new clients should prefer `/api/v1/backups/*`.
