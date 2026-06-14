---
order: 100
---

# Prometheus metrics

2FA-Vault exposes application and runtime metrics in Prometheus text exposition format at the `/metrics` endpoint. Scrape it with Prometheus, Grafana Agent, VictoriaMetrics, or any compatible collector.

## Enabling the endpoint

The `/metrics` endpoint is **disabled by default** and returns `404 Not Found` until you configure access. Enable it by setting either an IP allowlist or a bearer token (or both) via environment variables.

| Variable | Description |
|----------|-------------|
| `METRICS_ALLOWED_IPS` | Comma-separated list of IPs or CIDRs allowed to scrape `/metrics` without a token. Example: `192.168.1.0/24,10.0.0.5` |
| `METRICS_TOKEN` | Bearer token required when the request does not come from an allowed IP. Use a random string of at least 32 characters. |

A request is authorized when **either** condition is true:

- the source IP matches an entry in `METRICS_ALLOWED_IPS`, **or**
- the request carries `Authorization: Bearer <METRICS_TOKEN>`.

Configure them in your `.env`:

```bash !#
# Allow the scrape host's subnet, no token needed from there
METRICS_ALLOWED_IPS=192.168.1.0/24

# Fallback bearer token for scrapers outside the allowlist
METRICS_TOKEN=replace-with-a-long-random-secret
```

!!!warning Token strength
Generate `METRICS_TOKEN` with at least 32 random characters and rotate it if it is ever exposed. Treat it like a secret: do not commit it to source control.
!!!

## Available metrics

The endpoint exposes metrics such as:

| Metric | Type | Description |
|--------|------|-------------|
| `twofaccounts_total` | gauge | Total number of 2FA accounts on the instance |
| `http_requests_total` | counter | Total HTTP requests, labeled by status code |
| Additional runtime metrics | varies | Laravel/framework runtime gauges and counters |

Run `curl -H "Authorization: Bearer $METRICS_TOKEN" https://vault.example.com/metrics` to see the full, current output for your version.

## Example scrape config

Add a job to your Prometheus `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: '2fa-vault'
    metrics_path: /metrics
    scheme: https
    bearer_token: 'replace-with-a-long-random-secret'
    static_configs:
      - targets: ['vault.example.com']
```

If your Prometheus host is inside the `METRICS_ALLOWED_IPS` subnet, you can omit `bearer_token` and rely on the IP allowlist instead.

!!!info Behind a reverse proxy
When 2FA-Vault runs behind a reverse proxy (nginx, Traefik, Caddy), make sure the proxy forwards the real client IP. Configure [`TRUSTED_PROXIES`](/getting-started/config/env-vars/#trusted_proxies) and [`PROXY_HEADER_FOR_IP`](/getting-started/config/env-vars/#proxy_header_for_ip) so IP allowlist checks see the actual scraper address, not the proxy's.
!!!
