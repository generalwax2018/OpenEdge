# Security Features

## Principles

See [security-principles.md](security-principles.md) for encryption, Zero Trust (edge context), least privilege, defense in depth, secure defaults, and auditability.

## Built-in protections

- **Coraza WAF** with OWASP CRS (DetectionOnly by default) + request body limits
- **CrowdSec** collaborative threat intelligence + IP blocking
- **Distributed multi-zone rate limiting** via Redis
- **Security headers** (HSTS, CSP, nosniff, frame options, etc.)
- **Common attack path blocking**
- **TLS 1.2+ only**
- **Header size limits** (32 KiB)
- **Admin API disabled by default**
- **Optional Anubis bot challenge**
- **Shared Redis storage** for certificates and rate limits
- **Non-root containers** + `no-new-privileges`

## Origin protection (recommended)

Origins should not be directly reachable from the internet.

Options:
1. Private networking only (origin has no public IP; edge talks to it over VNet/VPC).
2. Firewall the origin to allow only the edge / load balancer IPs.
3. Shared secret header (uncomment `X-OpenEdge-Secret` in the Caddyfile and validate it on the origin).
4. Mutual TLS between edge and origin.

Prefer HTTPS upstreams (`https://...`).

## Encryption at rest

- Enable Redis AUTH and keep Redis private.
- Prefer Redis TLS if the network is not fully isolated.
- Store logs on encrypted volumes or ship them to an encrypted system.

## WAF request body limits

| Setting | Value | Purpose |
|---------|-------|---------|
| `SecRequestBodyLimit` | 10 MB | Max body inspected |
| `SecRequestBodyNoFilesLimit` | 128 KB | Max non-file body |
| `SecRequestBodyLimitAction` | Reject | Reject oversized bodies |
| `SecResponseBodyAccess` | Off | Skip response inspection by default |

## Rate limiting zones

| Zone | Path match | Limit |
|------|------------|-------|
| general | all | 120 req/min per IP |
| sensitive | `/login*`, `/admin*`, `/api/auth*`, etc. | 10 req/min per IP |

## Recommended hardening

1. Keep origins private or firewall them to the edge only.
2. Prefer HTTPS upstreams.
3. Enable DNS-01 for multi-instance.
4. Use strong random values for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`.
5. Keep Redis and CrowdSec LAPI private.
6. Start WAF in DetectionOnly, then switch to `On` after tuning.
7. Restrict DNS API tokens to minimum permissions.
8. Enable Anubis if bot traffic is high.
9. Rebuild the custom Caddy image periodically.

## Production checklist

See `docs/production-checklist.md`.
