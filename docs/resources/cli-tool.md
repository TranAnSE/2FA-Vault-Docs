---
order: 10
---

# 2fav CLI

The `2fav` command-line tool lets you interact with a 2FA-Vault instance from the terminal and from automation scripts. It is a standalone binary distributed from the [2FA-Vault-CLI](https://github.com/TranAnSE/2FA-Vault-CLI) repository.

The current release is the **Phase 1 MVP**, which covers authentication and read-only access to your accounts. End-to-end encryption unlock (Phase 2) is planned and is described at the bottom of this page.

## Installation

1. Download the binary for your platform from the [GitHub Releases](https://github.com/TranAnSE/2FA-Vault-CLI/releases) page (Windows, macOS, and Linux builds are provided).
2. Make it executable (Unix only) and place it on your `PATH`:
   ```bash !#
   chmod +x 2fav
   sudo mv 2fav /usr/local/bin/
   ```
3. Verify the installation:
   ```bash !#
   2fav --version
   ```

## Authentication

`2fav` authenticates with a [Personal Access Token (PAT)](/security/authentication/pat/). Never embed a PAT directly in shell scripts — use `2fav login` instead, which stores the token securely in your operating system credential store.

| OS | Credential store |
|----|------------------|
| macOS | Keychain |
| Windows | Credential Manager |
| Linux | libsecret / GNOME Keyring / KWallet (via `keyring`) |

### `2fav login`

```bash !#
2fav login --url https://vault.example.com --token YOUR_PAT
```

Stores the server URL and PAT in the OS credential store under a `2fav` entry. After a successful login, subsequent commands use the stored credentials automatically.

### `2fav logout`

```bash !#
2fav logout
```

Removes the stored credentials from the OS credential store. Run this on shared or ephemeral machines.

!!!warning Token security
The PAT is the same credential used by the REST API and grants the same access. Prefer `2fav login` (OS keychain) over passing `--token` on every command, which can leak the token through shell history and process listings.
!!!

## Commands

### `2fav list`

Lists your 2FA accounts.

```bash !#
2fav list
```

Returns the account ID, service, and account identifier for each entry.

### `2fav get`

Fetches a single account, matching by service name (case-insensitive substring) or numeric ID.

```bash !#
2fav get github
2fav get 42
```

### `2fav copy`

Generates the current OTP for an account and copies it to the clipboard.

```bash !#
2fav copy github
```

Useful for quick manual logins without leaving the terminal.

## CI / automation example

In a CI pipeline, log in once with the PAT (from a secret), then read OTPs in script steps:

```bash !#
2fav login --url "$VAULT_URL" --token "$VAULT_PAT"
DEPLOY_CODE=$(2fav get deploy-svc)
```

Store `$VAULT_PAT` as a protected, masked CI secret. Run `2fav logout` at the end of the job on self-hosted runners.

## Phase 2: end-to-end encryption

When vault encryption is enabled, the server cannot return plaintext OTP secrets. A future `2fav unlock` command (Phase 2) will derive the encryption key from your master password for the duration of the session, allowing `get` and `copy` to decrypt secrets locally.

Until Phase 2 ships, `2fav get` / `copy` fall back to the server-side OTP generation endpoint when available. Accounts that strictly require client-side decryption are reported as locked.

## Reference

- [2FA-Vault-CLI repository](https://github.com/TranAnSE/2FA-Vault-CLI)
- [Personal Access Tokens](/security/authentication/pat/)
- [REST API reference](/resources/rapidoc.html)
