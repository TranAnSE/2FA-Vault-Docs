---
order: 70
---
# Emergency access

Emergency access lets you designate trusted contacts who can request access to your vault if you become unavailable. It functions as a configurable dead man's switch.

Navigate to **Settings → Emergency Access** (`/settings/emergency`) to manage your contacts.

## Setting up a trusted contact

1. Enter the email address of a registered 2FA-Vault user.
2. Choose a **wait period**: 7, 14, 30, 60, or 90 days.
3. Choose an **access level**: view OTP only, or full access (can see decrypted secrets).
4. Click **Add**.

The contact receives a confirmation. Once they confirm, their status changes to _confirmed_.

You can designate up to **5 trusted contacts** per account.

!!!warning Security notice
Emergency access is a convenience–security trade-off. A trusted contact with full access can read your OTP secrets after the wait period. Choose contacts you trust completely and set an appropriate wait period to give yourself time to deny unauthorized requests.
!!!

## Requesting access (as a trusted contact)

Open **Settings → Emergency Access** and scroll to the **Vaults I can access** section. Click **Request access** next to the vault owner's entry. The owner is notified and has the configured wait period to approve or deny.

## Approving or denying a request

When a trusted contact requests access, a notification banner appears at the top of your emergency access settings page. You can:

- **Approve** — grants immediate access and optionally sends the encrypted vault key.
- **Deny** — rejects the request; the contact cannot request again immediately.

If you do not respond within the wait period, access is granted automatically.

## Dead man's switch

A scheduled task runs daily at 03:00 UTC. It checks all confirmed emergency contacts and grants access automatically when the vault owner has been inactive (no login) for longer than the configured wait period. This ensures access is available even if the owner is incapacitated.

## Revoking access

Click **Revoke** next to a contact at any time to remove their access immediately. Any active or future requests from that contact are cancelled.
