---
order: 86
---

# Breach monitoring

2FA-Vault can check the services and email addresses in your vault against [HaveIBeenPwned](https://haveibeenpwned.com) (HIBP) so you know when an account may have appeared in a known data breach.

There are two checks, with different privacy implications:

## Service / domain check

A **service check** looks up whether a service or domain (e.g. `github.com`) appears in HIBP's public breaches list. This uses only public data — **no opt-in is required**, and you can run it from any account via the **Check for breaches** button.

## Email check (opt-in)

An **email check** sends your email address to HIBP to see if that specific address appears in a breach. Because this sends personal data to a third party, it is **off by default** and hard-gated behind the **Email breach monitoring** preference.

To enable it:

1. Go to **Settings → Options → Breach monitoring**.
2. Toggle **Email breach monitoring** on.

Until the preference is enabled, the email-check endpoint refuses to run (HTTP 403) and no email is ever transmitted.

## How it works

- Results are cached for 24 hours per service and per email (the cache key is hashed, so the email is not stored in the cache in plaintext).
- HIBP rate limits and outages are handled gracefully — a transient failure degrades to an **unknown** status rather than an error, and the public service check keeps working even if no HIBP API key is configured.
- The HIBP API key (required only for email checks) is read from the `HIBP_API_KEY` environment variable and is never logged or returned to the client.

!!!warning Privacy
Email breach checks send your email address to HaveIBeenPwned. Enable the preference only if you accept that. Service/domain checks send only a public service name and need no opt-in.
!!!
