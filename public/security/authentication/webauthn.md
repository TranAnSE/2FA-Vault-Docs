# WebAuthn (aka Passkey)

2FA-Vault supports the W3C _Web Authentication_ API aka WebAuthn (<a href="https://webauthn.guide/" target="_blank">learn more</a>). This means you can register a security device like a Yubikey, a Titan Security Key, or a facial recognition system like Apple FaceId and use it to log into 2FA-Vault.

![image by Arun (dribbble.com/nullpointone)](/static/webauthn_login.gif)

This method is considered more secure, as it proves you are in fact you because you have to physically own the security device.

WebAuthn is available in 2FA-Vault as a complement or a replacement to the built-in login/password method. Consider using only WebAuthn to provide the highest protection to your 2FA-Vault instance.

!!!warning
The WebAuthn flow does not use login & password, but the creation of a user account with an email and a password remains mandatory.
!!!

---

## Registering a security device

2FA-Vault offers to register a WebAuthn device right after submitting the user registration form or through its _Settings > Webauthn_ section. Whatever you choose, the registration process will be the same and depends on the hardware you use (desktop, laptop or smartphone) and how your browser implements the WebAuthn flow.

A typical workflow would be:

1. You click the [!badge size="l" icon="plus-circle" text="Register a new device"] link in 2FA-Vault
2. Your browser prompts you to grant the operation
3. You put your finger on the key's touch button
4. 2FA-Vault registers the key and offers you to rename it

You can register several security devices, there is no limitation in the number of devices.

---

## Revoking a security device

You can revoke any registered security device through the _Settings > Webauthn_ section.  
Simply click the relevant [!badge variant="dark" text="Revoke"] button.

!!!danger
The revocation of a device is permanent and cannot be undone.
!!!

---

## I lost my device

Don't worry, there is always a solution, depending on how you have configured 2FA-Vault's WebAuthn options.  

+++ WebAuthn is the unique login method

If you have registered another device and still own it, just use this device to log in. Otherwise, you can recover your account by registering a new security device.

:icon-arrow-right: Click the __Recover your account__ link of the 2FA-Vault's login form, this will send a link to your registered email address. Follow this link, you will be able to register a new device and revoke all existing ones.

+++ WebAuthn is NOT the unique login method

Assuming you haven't lost your password too, switch the 2FA-Vault login form using the link __Sign in using login & password__ and log in using your email address and password. It's that simple!

!!!warning
Don't forget to revoke the lost device in the _Settings > Webauthn_ section.
!!!

+++

---

## Vault unlock with a passkey (PRF)

Since v1.7.0, a registered passkey with PRF support (hmac-secret extension) can also **unlock the E2EE vault** in addition to logging you in. See [Passkey unlock](/security/end-to-end-encryption/#passkey-unlock-webauthn-prf) for the full key-custody story.

1. Log in, **unlock the vault** (once), open _Settings → Security devices_.
2. Each passkey row shows an **Enable vault unlock** action; enrolled rows instead show an enabled badge with a **Refresh** action that re-runs the ceremony to replace the stored key material.
3. One user-verification gesture — the authenticator evaluates its PRF for the account salt in the same ceremony.
4. The unlock screen then shows an **Unlock with passkey** button next to the master-password form.

!!!warning
YubiKey firmware below 5.8 evaluates the PRF in two user-verification prompts during enrollment. This is normal for those keys and is accepted; the enrollment still completes.
!!!

---

## User verification

Most authenticators and smartphones will ask the user to actively verify themselves to log in. For example, through a touch plus pin code, password entry, or biometric recognition (e.g., presenting a fingerprint). The intent is to distinguish one user from any other.

Since v1.6.0 user verification is **always required** during the passkey login flow. Biometric devices (Touch ID, Windows Hello, face unlock) behave as before. Security keys **without a configured PIN** prompt for — or require — one, so make sure yours has a PIN set before upgrading to v1.6.0.
