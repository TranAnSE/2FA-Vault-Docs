---
order: 55
---
# Browser extension auto-fill

The 2FA-Vault browser extension can detect 2FA input fields on web pages and automatically fill the matching OTP code.

!!!warning Opt-in feature
Auto-fill is **disabled by default**. You must enable it explicitly in the extension settings. Only enable it on devices you trust.
!!!

## Enabling auto-fill

1. Open the extension popup and go to **Settings → Options**.
2. Scroll to the **Auto-Fill** section and toggle **Auto-fill OTP on web pages** on.

The extension will sync your account list to the background service worker the first time auto-fill is enabled.

## How it works

When you visit a page that contains an OTP or 2FA input field, the extension:

1. Detects qualifying input fields (fields with `autocomplete="one-time-code"`, names containing `otp`/`totp`/`2fa`, or `input[maxlength="6"]` with numeric input mode).
2. Matches the current domain against the `service` or `account` name of accounts in your vault.
3. If a match is found and the vault is unlocked, the OTP is fetched and inserted into the highest-confidence field.
4. A brief confirmation toast appears showing which account was used.
5. After **30 seconds**, the value is cleared from the input to reduce exposure.

Auto-fill is rate-limited to once per 30 seconds per page. If the vault is locked or no account matches the domain, nothing happens.

## Form detection badge

Even when auto-fill is disabled, the extension badge shows the number of matching accounts for the current page whenever a login or 2FA form is detected. Click the extension icon to open the popup and see the matched accounts highlighted.

## Limitations

- Auto-fill only works when the vault is **unlocked**.
- Sites with strict Content Security Policies that block content scripts will not be auto-filled. In those cases, copy the OTP from the popup manually.
- The domain matching is heuristic. If the wrong account is filled, use the extension popup to select the correct account manually.
- Auto-fill uses the server-side OTP endpoint (`GET /api/v1/twofaccounts/{id}/otp`). E2EE-encrypted accounts are not currently supported for auto-fill; the popup can still be used to copy the OTP manually for those accounts.

## Security considerations

- The OTP value is inserted into the page DOM and is therefore visible to scripts running on that page. Enable auto-fill only on trusted devices and on sites you trust.
- Auto-fill defaults to OFF specifically to ensure users make a conscious choice.
- The 30-second clear timer reduces the window during which the OTP value sits in a form field.
