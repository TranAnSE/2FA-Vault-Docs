# Sessions & tokens hardening (v1.3.1)

The v1.3.1 workflow-audit release hardened how sessions and API tokens are
managed. This page describes the operator- and user-visible changes.

## Session eviction now works

- The recommended session driver is now **`database`** (set in `.env.example`).
  With it, revoking a session from _Settings → Sessions_ deletes the real
  server-side session row; the next request carrying that cookie is rejected.
- On the legacy `file` driver the revocation list is still shown, but eviction
  of browser sessions is not possible — consider switching drivers.
- Changing or resetting a password (self-service or admin-triggered) now
  **revokes every personal access token, evicts sessions and re-arms the
  vault lock**. This is the standard incident-response playbook for a
  compromised account.
- Deactivated users (admin user management) are now blocked everywhere:
  existing sessions are terminated and API requests return `401`.

## Personal access token scopes

- New PATs **must request at least one scope**: `read`, `otp`, `write` or
  `admin`. Omitting scopes is rejected with `422`.
  - The browser extension works best with `read` + `otp` (+ `write` if you
    let it sync accounts); the CLI needs `read` + `otp`.
- New PATs expire after **90 days** by default (previously 1 year).
- The token list in _Settings → OAUTH_ now shows each token's expiration date,
  so an approaching 90-day expiry is visible before a token stops working.
- PATs created before this release were stamped with an internal
  `legacy_full_access` marker by a one-time migration and keep working
  unchanged. **Rotate them at your convenience** by creating scoped
  replacements and revoking the old ones.

## Backup format v2

- Backups exported from the web app are now **encrypted in the browser** with
  your backup password (Argon2id + AES-GCM) before download — the server
  never sees the password.
- Automatic backup destinations can define a per-destination
  **encryption password**; pushed files are then encrypted before they reach
  S3/WebDAV. Email attachments are only sent when explicitly opted in, and
  only in encrypted form.
- Importing: format-2 files are decrypted in the app before upload. An
  undecrypted format-2 upload is rejected — there is no silent legacy
  fallback. Legacy (format-1/plain) files still import with a warning.
  After an import that contains encrypted accounts, the app tries to decrypt
  them and offers a one-click cleanup if the master password does not match.

## Emergency access (repaired)

The dead man's switch is now cryptographically functional:

- When you designate a registered contact, your **master password is wrapped
  in your browser with the contact's RSA public key** and stored alongside
  the contact. When (and only when) the switch triggers or you approve a
  request, the grantee can read your encrypted accounts and unwrap the key
  locally — read-only.
- If the wrapped key is missing or the grantee rotated their key, the vault
  access endpoint **fails closed** (409) and both sides are told to re-save
  the contact. Watch for the "key missing or outdated" badge on the
  Emergency access settings page after rotating keys or master passwords.
