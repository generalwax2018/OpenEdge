# Security Features

## Built-in protections

- **Coraza WAF** with OWASP CRS (DetectionOnly by default) + request body limits
- **CrowdSec** collaborative threat intelligence + IP blocking
- **Distributed multi-zone rate limiting** via Redis
- **Security headers** (HSTS, CSP, nosniff, frame options, etc.)
- **Common attack path blocking** (`.env`, `.git`, `wp-admin`, `phpmyadmin`, etc.)
- **TLS 1.2+ only** (explicit)
- **Header size limits** (32 KiB)
- **Optional Anubis bot challenge**
- **Shared Redis storage** for certificates and rate limits
- **Non-root containers** + `no-new-privileges`

## WAF request body limits

| Setting | Value | Purpose |
|---------|-------|---------|
| `SecRequestBodyLimit` | 10 MB | Max body inspected |
| `SecRequestBodyNoFilesLimit` | 128 KB | Max non-file body |
| `SecRequestBodyLimitAction` | Reject | Reject oversized bodies |
| `SecResponseBodyAccess` | Off | Skip response inspection by default |

These reduce the risk of memory exhaustion from large or malicious requests. Adjust upward if your app needs larger uploads, then re-test.

## Rate limiting zones

| Zone | Path match | Limit |
|------|------------|-------|
| general | all | 120 req/min per IP |
| sensitive | `/login*`, `/admin*`, `/api/auth*`, etc. | 10 req/min per IP |

## Bot challenge (Anubis)

Optional proof-of-work challenge. Enable with:

```bash
UPSTREAM=http://anubis:3000
ANUBIS_TARGET=http://your-origin:80
docker compose --profile bot-challenge up -d --build
```

## Central CrowdSec

Point all nodes at one LAPI:

```
CROWDSEC_LAPI_URL=http://central-crowdsec.example.com:8080
CROWDSEC_BOUNCER_KEY=key-registered-on-central
```

## Recommended hardening

1. Enable DNS-01 challenges for multi-instance (see `docs/dns-challenge.md`).
2. Use strong random values for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`.
3. Keep Redis and CrowdSec LAPI on private networks only.
4. Uncomment `admin off` in the Caddyfile for production.
5. Start WAF in DetectionOnly, review logs, then switch to `On`.
6. Restrict DNS API tokens to minimum permissions.
7. Enable Anubis if you see heavy bot traffic.
8. Rebuild the custom Caddy image periodically.
9. Tune `SecRequestBodyLimit` if you need larger uploads.

## Production checklist

See `docs/production-checklist.md`.
