---
order: 1001
---

# Email invitations

[!badge variant="warning" text="Admin only"]

Email invitations let an administrator invite a specific person to create an account by sending them a single-use invitation link. This is the recommended way to onboard users when public registration is restricted or disabled.

## When to use invitations

Invitations are useful when:

- [Registration is restricted or disabled](/usage/administration/#registration-control), but you still want to admit selected users.
- You want a known identity (the invitee's email) tied to the new account.
- You want to grant the new user a specific role at invitation time.

## Sending an invitation

1. Go to **Admin → Users → Invite user**.
2. Enter the recipient's **email address**.
3. Choose a **role** for the new account (e.g. _member_, _admin_).
4. Click **Send invitation**.

2FA-Vault emails the recipient a link valid for **7 days**. The link skips the normal registration-open check, so the invitee can register even when public registration is disabled.

## How invitees register

The invitee clicks the link in the email and lands on the registration form with the email field pre-filled. After they complete registration, the invitation is marked _accepted_ and can no longer be reused.

A single invitation creates exactly one account. An invitation that has been used cannot be re-used, and an expired invitation cannot be redeemed.

## Managing pending invitations

The invitation list at **Admin → Users → Invitations** shows every pending invitation with its status:

| Status | Meaning |
|--------|---------|
| _Pending_ | Sent, not yet used, not expired |
| _Expired_ | The 7-day validity window has elapsed |
| _Accepted_ | The invitee registered successfully |

To cancel an outstanding invitation before it is used, click **Revoke**. The link is invalidated immediately and can no longer be used to register. Revoking is useful when an invitation was sent to the wrong address or is no longer needed.

!!!info Email configuration required
Invitations are sent by email, so a valid [email configuration](/getting-started/config/env-vars/#email-setting) must be set. You can verify delivery with the [email test tool](/usage/administration/#email-testing) under _Admin > App Setup_.
!!!
