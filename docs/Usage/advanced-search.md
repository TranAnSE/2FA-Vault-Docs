---
order: 88
---
# Advanced search & filter

The accounts view supports full-text search and multi-criteria filtering to help you find accounts quickly even in large vaults.

## Search

Type in the search box to filter accounts by service name, account name, or tag name. Search terms are matched as substrings and are case-insensitive. Multiple words narrow the results (implicit AND).

When vault encryption is enabled, search runs entirely in the browser against the already-decrypted account list — no search terms are sent to the server.

When encryption is disabled, the API accepts a `q` query parameter for server-side LIKE search across `service` and `account` columns.

## Filter panel

Click the filter icon in the toolbar to open the filter panel. Available filters:

| Filter | Values |
|--------|--------|
| OTP type | TOTP, HOTP, Steam TOTP |
| Algorithm | SHA-1, SHA-256, SHA-512 |
| Digits | 6, 7, 8 |
| Group | Any group in your vault |
| Tags | One or more tags (OR / AND mode) |
| Encryption | All / Encrypted / Not encrypted |

## Sorting

Results can be sorted by:

- **Default order** — custom drag-and-drop order
- **Service name** (A → Z)
- **Account name** (A → Z)
- **Date added** (newest first)
- **Last used** (most recent first)

## Saved presets

Click **Save preset** at the bottom of the filter panel and enter a name to store the current filter and sort configuration. Presets are stored in `localStorage` and restored instantly when selected. You can delete individual presets from the same panel.

## API query parameters

Server-side filtering is available on `GET /api/v1/twofaccounts`:

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | Full-text search (service, account) |
| `types` | string | Comma-separated OTP types: `totp,hotp,steamtotp` |
| `algorithms` | string | Comma-separated: `sha1,sha256,sha512` |
| `digits` | string | Comma-separated: `6,7,8` |
| `group_id` | integer | Filter by group ID |
| `tags` | string | Comma-separated tag IDs |
| `tag_mode` | string | `and` or `or` (default `or`) |
| `encrypted` | boolean | Filter by encryption status |
| `last_used_from` | ISO8601 | Earliest last-used timestamp |
| `last_used_to` | ISO8601 | Latest last-used timestamp |
| `sort` | string | `service`, `account`, `created_at`, `last_used_at`, `order_column` |
| `sort_dir` | string | `asc` or `desc` |
