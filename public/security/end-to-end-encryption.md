# End-to-end encryption

2FA-Vault adds a zero-knowledge vault encryption flow on top of the upstream 2FA-Vault security model.

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

## Passkey unlock (WebAuthn PRF)

Since v1.7.0 the vault can be unlocked with a **PRF-capable passkey** (WebAuthn hmac-secret / prf extension) as an alternative to typing the master password. The master-password path always remains available as a fallback — the passkey is an *alternative*, not a replacement.

### How it works (key custody, honest version)

1. At enrollment the browser asks the authenticator to evaluate its PRF (hmac-secret) for a random per-user salt, stretches that output with HKDF-SHA256 into an AES-256-GCM wrapping key, generates a fresh **RSA-2048 keyset**, and:
   - wraps the RSA **private key** under the PRF-derived wrapping key,
   - encapsulates the vault key K under the RSA public key.
2. Only the three opaque blobs travel to the server (`public_key`, `wrapped_private_key`, `wrapped_vault_key`) — the PRF output and the wrapping key never leave the authenticator/browser, and the server never sees the vault key in the clear.
3. At unlock, the authenticator re-evaluates the PRF for the stored salt (one UV gesture), the browser unwraps the private key and decapsulates K, verifies it against the encryption test value, and clears everything transient from memory when the vault locks again.
4. On master-password rotation the client re-encapsulates the new vault key under each enrolled passkey's public key — **no passkey ceremony** (public-key-only operation).

The server stores an opaque blob triple per passkey plus a per-user PRF salt. The blobs are only as recoverable as the passkey itself: **if the passkey is deleted, or the authenticator is lost without a backup, passkey unlock for that credential is gone** — the master password remains the recovery path.

### Enrolling and using passkey unlock

Enrollment is in **Settings → Security devices** (per-passkey "Enable vault unlock" action; the vault must be unlocked). The unlock screen shows an **Unlock with passkey** button whenever at least one enrollment exists. After a master-password change, existing enrollments are re-wrapped automatically at rotation time; if that re-wrap fails (e.g. a racing tab rotated first), Settings → Encryption shows a refresh banner with a one-click retry.

### Requirements and limitations

- The authenticator must support the WebAuthn **prf** extension (most platform authenticators — Touch ID, Windows Hello, Android — and recent security keys do; YubiKey firmware < 5.8 keys show a second touch prompt during enrollment — documented, accepted).
- The passkey must be registered for the account (Settings → Security devices) before it can be enrolled for vault unlock.
- SSO-only instances deny the PRF endpoints (the same `manage-webauthn-credentials` gate as WebAuthn management); the password path is unaffected.
- The passkey unlock button stays hidden until an enrollment exists; the password form is always visible.
- After a master-password **reset** (device-lost recovery flow), all passkeys and their enrollments are flushed — passkey unlock must be re-enrolled.

## Key sharing for teams

Team owners can share encrypted OTP secrets with members without the server ever learning the plaintext secret. Each user maintains an RSA-2048 key pair: the public key is registered on the server; the private key stays only in the user's browser (IndexedDB).

When sharing, the owner:
1. Decrypts the account secret locally using their vault key.
2. Fetches each recipient member's public key from the server.
3. Encrypts the secret with RSA-OAEP using each member's public key.
4. Uploads the per-member wrapped keys via `POST /api/v1/teams/{id}/share-encrypted`.

When a member views shared accounts, they receive their personal `wrapped_key` and decrypt it locally with their private key to generate OTPs. The server stores only opaque ciphertext.

See [Teams and encrypted backups](/usage/teams-and-backups/) for the full workflow.

## Legacy database encryption

2FA-Vault still documents the upstream database encryption option separately. Legacy DB encryption depends on `APP_KEY` and protects fields at rest on the server. E2EE is different: account secrets are encrypted before they reach the server.
