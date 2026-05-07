---
icon: package
---
# API

2FA-Vault is built on top of its own REST API (following OpenAPI 3.1 specification), which can be used to make other applications communicate with a 2FA-Vault instance.

The API provides endpoints to manage most 2FA-Vault resources:

Resource   | Description { class="compact" }
---    | ---
twofaccounts | The 2FA accounts stored in 2FAuth which you need to generate One-Time Passwords (OTP)
one-time password | The One-Time Passwords (TOTP or HOTP) generated on demand
groups | The groups used to organize 2FA accounts in 2FAuth
qrcode | Two-dimensional barcode used to encode/share 2FA accounts
icons | Images used to illustrate 2FA accounts in 2FAuth
preferences | The user preferences
settings | The 2FAuth administrator settings, which can be extended with custom settings
encryption | End-to-end encryption setup, status, lock, unlock, and disable metadata endpoints
backups | Encrypted backup export, import, metadata, and statistics endpoints
push | Web push subscription and test-notification endpoints
teams | Team, invitation, member, and account-sharing endpoints
features | Runtime feature-flag discovery endpoints

---

## Authentication

You authenticate in the 2FA-Vault API with a _Personal Access Token_ (PAT) built upon the OAUTH `Bearer` authentication scheme (see <a href="https://datatracker.ietf.org/doc/html/rfc6750" target="_blank">RFC 6750</a>).  
That means the PAT has to be passed via the HTTP `Authorization` header in every request made to the API.

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJhdWQiOiIxIiwianRpIjoiMzZjOTc5NmFlZGI2OGQyYmE2YTIyMTE0NTN
```

A PAT is valid until you decide to revoke it.

### Creating an access token

Open the 2FA-Vault _Settings > OAUTH_ section and click the [!badge size="l" icon="plus-circle" text="Generate a new token"] link to generate a new token.

![A PAT (in green) right after its creation](/static/personal_access_token.png)

!!!warning
The token will only be shown __once__, right after its creation, so copy it __immediately__ because you won't be able to display it again.
!!!

### Revoking a token

You can revoke a personal access token by simply clicking its [!badge variant="dark" text="Revoke"] button in the _Settings > OAUTH_ section. A request made with a revoked token will receive a `401 Unauthorized` response.

!!!danger
The revocation of a token is permanent and cannot be undone.
!!!

---

## Fork-specific endpoints

2FA-Vault extends the upstream API with routes used by the current web app and browser extension:

- `GET /api/v1/twofaccounts/encrypted` returns encrypted account payloads for clients that can decrypt locally.
- `PATCH /api/v1/twofaccounts/{id}/counter` advances an HOTP counter without touching the encrypted secret.
- `GET /api/v1/twofaccounts/count` returns the authenticated user's account count.
- `/api/v1/encryption/*` stores and reads client-generated E2EE metadata. The server never receives a master password or derived vault key.
- `/api/v1/backups/*` handles `.vault` backup export, import, metadata preview, and backup stats.
- `/api/v1/push/*` manages browser/PWA web push subscriptions.
- `/api/v1/teams/*` manages teams, invitations, members, and shared accounts.
- `/api/v1/features/*` exposes runtime feature flags.

The browser extension uses the encrypted account and encryption metadata endpoints to unlock the vault locally and generate OTPs without server-side decryption.

---

## API documentation

The API has its own dedicated documentation that you can browse in a lightweight format below.  
You may also use the fullscreen format which provides previous versions, a more comfortable layout and modern features like advanced search, mocking and more:

[!ref target="blank" icon="code-square" text="Fullscreen documentation"](/resources/rapidoc.html)

[!embed](/resources/rapidoc-embeded.html)
