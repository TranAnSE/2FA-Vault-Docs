---
order: 80
---
# End-to-end encryption

2FA-Vault adds a zero-knowledge vault encryption flow on top of the upstream 2FAuth security model.

In this mode, the browser or browser extension derives a vault key from the user's master password with Argon2id, then encrypts OTP secrets with AES-GCM before sending account data to the server. The server stores only metadata and ciphertext:

- encryption salt
- encrypted verification payload
- encryption version
- vault lock state
- encrypted account payloads

The server must not receive the master password, derived vault key, or plaintext OTP secret for encrypted records.

## Setup and unlock

During setup, the client sends `encryption_salt`, `encryption_test_value`, and `encryption_version` to `/api/v1/encryption/setup`.

During unlock, the client reads `/api/v1/encryption/info`, derives the key locally, decrypts the verification payload locally, and then posts only the boolean verification result to `/api/v1/encryption/verify`.

## Encrypted account sync

Encrypted accounts are available through `GET /api/v1/twofaccounts/encrypted`. This endpoint returns account metadata plus the raw ciphertext JSON string in the `secret` field. It is intended for clients such as the browser extension that can decrypt locally after unlock.

HOTP accounts can advance their counter through `PATCH /api/v1/twofaccounts/{id}/counter` without rewriting or revalidating the encrypted secret.

## Browser extension behavior

The forked browser extension can unlock an encrypted vault using the same metadata as the web app. After unlock, it decrypts account payloads locally and generates OTPs in the popup session. Locking the extension clears the derived key and decrypted account data from the extension session.

## Legacy database encryption

2FA-Vault still documents the upstream database encryption option separately. Legacy DB encryption depends on `APP_KEY` and protects fields at rest on the server. E2EE is different: account secrets are encrypted before they reach the server.
