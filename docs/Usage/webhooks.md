---
order: 60
---
# Webhooks

Webhooks let you receive HTTP POST notifications when events occur in your vault. Use them to trigger automation in n8n, Zapier, custom scripts, or any HTTP-capable service.

Navigate to **Settings → Webhooks** (`/settings/webhooks`) to manage your webhooks.

## Creating a webhook

1. Click **Create webhook**.
2. Enter a **name** (for your reference), the **endpoint URL**, and an optional **signing secret**.
3. Select the **events** you want to subscribe to.
4. Click **Save**.

## Signing secret

If you provide a signing secret, every delivery includes an `X-2FA-Vault-Signature` header containing `sha256=<HMAC-SHA256 of the raw request body>`. Verify this header in your endpoint to confirm the request is genuine.

## Event types

| Event | Triggered when |
|-------|---------------|
| `account.created` | A new 2FA account is added |
| `account.updated` | An account's details are changed |
| `account.deleted` | An account is deleted |
| `account.otp_generated` | An OTP is generated for an account |
| `team.created` | A new team is created |
| `team.deleted` | A team is deleted |
| `team.member_joined` | A user joins a team |
| `team.member_left` | A user leaves a team |
| `team.member_removed` | A member is removed by an admin |
| `team.account_shared` | An account is shared with a team |
| `team.account_unshared` | A shared account is removed |
| `auth.user_login` | A user logs in |
| `vault.locked` | A vault is locked |
| `vault.unlocked` | A vault is unlocked |

## Payload format

```json
{
  "event": "account.created",
  "timestamp": "2026-06-07T12:34:56Z",
  "data": { ... }
}
```

The `data` object contains event-specific fields. Account events include `id`, `service`, and `account` (never the OTP secret). Team events include `team_id` and `team_name`.

## Delivery and retry

Webhooks are delivered asynchronously via the Laravel queue. If the endpoint returns a non-2xx response or times out (10 second limit), delivery is retried up to **3 times** with a 60-second delay between attempts. After 3 failures the delivery is marked as failed.

## Testing a webhook

Click **Test** next to any webhook to send a `webhook.test` payload immediately. The delivery appears in the webhook's history.

## Delivery history

Click **History** next to a webhook to see the last 50 deliveries with event name, HTTP status code, and delivery timestamp. Failed deliveries are highlighted in red.

## Enabling and disabling

Each webhook can be toggled on or off with the **Enable** / **Disable** button. Disabled webhooks do not receive deliveries.
