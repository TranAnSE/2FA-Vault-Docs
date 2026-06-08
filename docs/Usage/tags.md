---
order: 90
---
# Tags & labels

Tags let you label accounts with colour-coded chips so you can organise them across groups or filter them flexibly.

## Managing tags

Open **Settings → Tags** (or navigate to `/tags`) to create, rename, recolour, and delete tags. Each tag has a name (up to 100 characters) and a hex colour value. Deleting a tag removes it from every account automatically.

Tags are scoped per user. Other users on the same instance cannot see or use your tags.

## Assigning tags to accounts

When creating or editing an account, the **Tags** field shows an autocomplete input. Start typing to search existing tags or press Enter to select. Multiple tags can be assigned to a single account (up to 10). Remove a tag from an account by clicking the × on its chip.

Tag assignments can also be updated via the API:

```
POST /api/v1/twofaccounts/{id}/tags
Body: { "tags": [1, 3, 7] }
```

This endpoint replaces the entire tag set for the account.

## Filtering by tag

The filter panel in the accounts view includes a **Tags** section. Click any tag chip to toggle it. Two filter modes are available:

- **OR** (default) — show accounts that have at least one of the selected tags.
- **AND** — show only accounts that have all selected tags.

The selected tags and mode can be saved as a named preset so you can restore the filter in one click.

## API reference

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/tags` | List all tags (with account count) |
| POST | `/api/v1/tags` | Create a tag `{name, color?}` |
| PUT | `/api/v1/tags/{id}` | Rename or recolour a tag |
| DELETE | `/api/v1/tags/{id}` | Delete tag (removed from all accounts) |
| POST | `/api/v1/twofaccounts/{id}/tags` | Sync tags for an account `{tags: [id]}` |
