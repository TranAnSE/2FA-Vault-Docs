---
order: 75
---
# Multiple vaults

2FA-Vault lets you create up to **10 independent vaults**, each with its own master password and E2EE encryption key. Common uses include separating personal, work, and client accounts.

Navigate to **Settings → Vaults** (`/settings/vaults`) to manage your vaults.

## How vaults work

Each vault is a fully isolated container. Accounts, groups, and tags belong to exactly one vault. When you switch to a different vault you may be prompted to unlock it if it has E2EE enabled and is currently locked.

Your first vault is created automatically when you enable E2EE. It is marked as the **default vault** and cannot be deleted.

## Creating a vault

1. Open **Settings → Vaults**.
2. Enter a name for the new vault.
3. Click **Create**.

The vault is created unlocked and without encryption. To encrypt it, use the vault encryption setup flow in **Settings → Encryption** after selecting the vault.

## Renaming a vault

Click **Rename** next to a vault name, enter the new name, and press **Save**.

## Locking a vault

Click **Lock** to immediately clear the vault key from memory. The vault enters a locked state and its accounts cannot be viewed until you unlock it again. The default vault lock behaviour follows the same auto-lock rules as the main encryption settings.

## Deleting a vault

Non-default vaults can be deleted. All accounts, groups, and tags inside the vault are permanently removed. This action cannot be undone.

!!!danger
Deleting a vault permanently deletes all accounts it contains. Export the vault first if you need to keep the data.
!!!

## API reference

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/vaults` | List vaults with account and group counts |
| POST | `/api/v1/vaults` | Create a vault `{name}` |
| PUT | `/api/v1/vaults/{id}` | Rename a vault `{name}` |
| DELETE | `/api/v1/vaults/{id}` | Delete a non-default vault |
| POST | `/api/v1/vaults/{id}/lock` | Lock a vault |
| POST | `/api/v1/vaults/{id}/encryption` | Set vault encryption salt and test value |
