---
order: 85
---
# Vault health dashboard

The vault health dashboard gives administrators a quick summary of the security quality of the vault. All calculations run entirely in the browser — no account secrets or plaintext data are sent to the server.

Navigate to **Admin → Vault Health** or `/admin/health`.

## Overall health score

The dashboard computes a weighted score from 0 to 100:

| Metric | Weight | What is measured |
|--------|--------|-----------------|
| Duplicate secrets | 25 % | Accounts sharing the same OTP secret |
| Unused accounts | 20 % | Accounts not used to generate an OTP in the past 90 days |
| Weak secrets | 30 % | Accounts with low entropy, SHA-1 algorithm, or 6-digit codes |
| Incomplete data | 15 % | Accounts missing a service name or icon |
| Ungrouped accounts | 10 % | Accounts not assigned to any group |

The score decreases for each flagged account. A score of 100 means no issues were found.

### Colour scale

| Score | Colour | Meaning |
|-------|--------|---------|
| 76 – 100 | Green | Good |
| 51 – 75 | Yellow | Fair |
| 26 – 50 | Orange | Warning |
| 0 – 25 | Red | Critical |

## Finding details

Each metric has an expandable section that lists the affected accounts. The dashboard does not reveal raw secret values — only account names, service names, and relevant metadata are shown.

## Exporting a report

Click **Export report** to download a JSON file containing the health score, metric breakdown, and lists of unused and incomplete accounts. The export file never includes OTP secrets.

!!!info Last-used tracking
The `last_used_at` timestamp for an account is updated every time an OTP is generated via the API (`GET /api/v1/twofaccounts/{id}/otp`). Accounts created before v1.1.0 will have a null `last_used_at` and will appear in the unused list until their first OTP is generated.
!!!
