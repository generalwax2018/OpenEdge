# Security Features

## Built-in protections

- **Coraza WAF** with OWASP CRS (DetectionOnly by default)
- **CrowdSec** collaborative threat intelligence + IP blocking
- **Distributed multi-zone rate limiting** via Redis
- **Security headers** (HSTS, CSP, nosniff, frame options, etc.)
- **Optional Anubis bot challenge** (proof-of-work against scrapers/AI crawlers)
- **Shared Redis storage** for certificates and rate limits
- **Non-root containers** + `no-new-privileges`

## Rate limiting zones

| Zone | Path match | Limit |
|------|------------|-------|
| general | all | 120 req/min per IP |
| sensitive | `/login*`, `/admin*`, `/api/auth*`, `/wp-login*`, `/xmlrpc.php` | 10 req/min per IP |

Limits are shared across all instances when using Redis.

## Bot challenge (Anubis)

Optional proof-of-work challenge that sits between Caddy and your origin. Useful against AI scrapers and automated bots.

Enable:

```bash
# In .env
UPSTREAM=http://anubis:3000
ANUBIS_TARGET=http://whoami:80   # or your real origin
ANUBIS_DIFFICULTY=4

# Start with the profile
docker compose --profile bot-challenge up -d --build
```

Anubis issues a cookie after a successful challenge so legitimate browsers are not repeatedly challenged.

## Central CrowdSec

By default each stack runs its own CrowdSec. For multi-instance you can point all nodes at one central LAPI:

```
CROWDSEC_LAPI_URL=http://central-crowdsec.example.com:8080
CROWDSEC_BOUNCER_KEY=the-key-registered-on-the-central-lapi
```

Register the bouncer key on the central CrowdSec with `cscli bouncers add openedge`.

## Recommended hardening

1. Enable DNS-01 challenges (see `docs/dns-challenge.md`).
2. Use strong random values for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`.
3. Keep Redis and CrowdSec LAPI on private networks only.
4. Uncomment `admin off` in the Caddyfile for production.
5. Start WAF in DetectionOnly, review logs, then switch to `On`.
6. Restrict DNS API tokens to minimum permissions.
7. Enable Anubis if you see heavy bot traffic.
8. Rebuild the custom Caddy image periodically.

## Production checklist

See `docs/production-checklist.md`.
