---
label: Welcome
---
# Welcome to 2FA-Vault Docs

> 2FA-Vault is a personal fork of 2FA-Vault. It is a self-hosted OTP vault for desktop, mobile web, PWA usage, and a browser extension, with fork-specific work around end-to-end encryption, teams, encrypted backups, and web push.

<style>
    html.dark .light-screen,
    html:not(.dark) .dark-screen {
        display: none;
    }
    html.dark .dark-screen,
    html:not(.dark) .light-screen {
        display: block;
    }
</style>
:::dark-screen
![Screenshots of 2FA-Vault on mobile](/static/2fauth_screenshots_dark.png)
:::
:::light-screen
![Screenshots of 2FA-Vault on mobile](/static/2fauth_screenshots_light.png)
:::

---

## Why 2FA-Vault

Two-Factor Authentication has become very popular in recent years, resulting in more and more situations where we face a security code request and an increase in the number of accounts protected by this technology. In other words, 2FA is now inevitable and critical.

2FA-Vault's purpose is to simplify how you use and manage your 2FA with a clean web interface while keeping the deployment model familiar for existing 2FA-Vault users.

Moreover, as an open source and self-hosted application, it lets you regain control over your personal security data, giving you privacy and the ability to back it up (Have you lost a smartphone with all your 2FA accounts inside Google Auth? I did... it really sucked)

!!!warning Fork status
This documentation targets the 2FA-Vault fork, not the official 2FA-Vault release. Upstream 2FA-Vault behavior still applies where unchanged, but fork-only features such as E2EE, team sharing, encrypted backups, web push, and encrypted extension sync require a 2FA-Vault instance.
!!!

## Features

#### :icon-ellipsis: Generate passwords

The main purpose of 2FA-Vault: Serve you some fresh TOTP/HOTP security codes aka One-Time Passwords.

#### :icon-device-desktop: Work anywhere

It's a Web App, it just works, whatever device you're on. You only need one device (not even yours) and an Internet connection.

#### :icon-apps: QR codes scan

Scan and decode QR codes to add a 2FA account in no time. Actually, it decodes any QR code, even non 2FA.

#### :icon-database: 2FA management

Manage your 2FA accounts, organize and classify them using Groups, edit & delete them. You can even manually add an account without scanning a QR code.

#### :icon-shield-check: Protect your data

2FA-Vault protects your data with privacy-first self-hosting, WebAuthn authentication, OTP obfuscation, auto lock, optional legacy database encryption, and fork-specific end-to-end encryption.

#### :icon-people: Multi-user

Share your instance with your family, your friends. Everyone can have an account.

#### :icon-organization: Teams

Create teams, invite members, assign roles, and share selected accounts with explicit access levels.

#### :icon-lock: End-to-end encryption

When vault encryption is enabled, clients derive keys locally with Argon2id and encrypt secrets with AES-GCM. The server stores salt, encrypted verification payloads, vault lock state, and ciphertext, but not master passwords or derived keys.

#### :icon-arrow-switch: Import / Export

Migrate from another 2FA app to 2FA-Vault or export your 2FA data in a breeze.

#### :icon-tag: Tags & advanced search

Label accounts with colour-coded tags and filter your vault by OTP type, algorithm, group, tag combination, or last-used date. Filters can be saved as named presets.

#### :icon-pulse: Vault health

The admin vault-health dashboard scores your vault 0–100, highlights duplicate secrets, flags accounts unused for 90+ days, and rates secret strength by entropy and algorithm.

#### :icon-share: Encrypted key sharing

Team owners can share encrypted OTP secrets with individual members using RSA-OAEP key wrapping. The server stores only ciphertext — it never sees plaintext secrets or derived keys.

#### :icon-shield-check: Biometric unlock

Enable fingerprint or Face ID unlock for the web app and browser extension using the WebAuthn platform authenticator. The encrypted master key is stored locally; biometric authentication is required to decrypt it.

#### :icon-person: Emergency access

Designate trusted contacts who can request access to your vault. You choose the wait period (7–90 days) and access level. If you do not respond within the window, access is granted automatically.

#### :icon-stack: Multiple vaults

Create up to ten independent vaults, each with its own master password and E2EE key. Switch vaults to separate personal, work, or client accounts.

#### :icon-clock: Team activity log

Every team action — member join/leave, account share/unshare, role change — is recorded in a searchable activity log. Owners and admins can export the log as JSON.

#### :icon-webhook: Webhooks

Subscribe to account, team, and auth events and receive signed HTTP POST notifications. Supports retry with exponential back-off and tracks delivery history per endpoint.

#### :icon-browser: Browser auto-fill

The browser extension detects 2FA input fields on web pages, matches the domain against your vault, and fills the OTP automatically. Auto-fill is opt-in and clears the code from the page after 30 seconds.

## REST API

2FA-Vault provides a REST API which lets you perform most of its functionalities from any external application. Have a look at the [API documentation](/api/) to find out how to use it.

## Browser extensions

By design, 2FA-Vault is always at your fingertips through a pinned tab, shortcuts, PWA installation, or browser extension. Browser extensions complement this feature by offering OTP generation directly from your browser toolbar.

When vault encryption is enabled, the extension now unlocks against your vault metadata and reads encrypted account payloads without requiring server-side decryption. The derived vault key is stored only in browser session storage when available and is cleared again when the extension locks.

Note: You must have a running 2FA-Vault instance to use the forked extension; it is not standalone.

<figure class="content-left">
    <a href="https://chromewebstore.google.com/detail/2FA-Vault-beta/kokhpbhfeokchmbimdlaldcmlinjpipm" target="_blank">
        <img src="static/available_on_chrome_web_store_bordered.png" alt="Available on Chrome web store"/>
    </a>
</figure>
<figure class="content-left">
    <a href="https://addons.mozilla.org/fr/firefox/addon/2FA-Vault-addon/" target="_blank">
        <img src="static/available_on_amo.webp" alt="Get the Firefox addon"/>
    </a>
</figure>
