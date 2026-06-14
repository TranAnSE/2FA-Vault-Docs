---
order: 77
---

# Session management

The session manager shows every active login session for your account and lets you revoke any of them — including on devices you no longer have access to.

## Viewing active sessions

Go to **Settings → Sessions** to see all of your active sessions. Each session is listed with:

- The IP address it originated from
- The client user agent (browser and operating system)
- When the session was created
- When it was last active

Your current session is highlighted so you can tell it apart from the others.

## Revoking a session

Click **Revoke** next to a session to end it immediately. The affected device is logged out on its next request and must authenticate again.

- Revoking **another** session does not affect your current login.
- Revoking your **current** session logs you out of 2FA-Vault immediately.

Use this after signing in on a shared or public computer, or any time you suspect a session should no longer be trusted.

!!!warning Personal Access Tokens are separate
Session management covers browser login sessions only. Long-lived API credentials are [Personal Access Tokens](/security/authentication/pat/) and are managed separately under _Settings → Personal Access Tokens_ (or revoked by an administrator from _Admin > Users_).
!!!
