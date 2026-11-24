# Security Notes

## Built-in protections

- **Coraza WAF** with OWASP CRS (starts in DetectionOnly — switch to On after tuning)
- **CrowdSec** collaborative threat intelligence and IP blocking
- **Distributed rate limiting** via Redis
- **Security headers** (HSTS, nosniff, frame options, etc.)
- **Non-root containers** and `no-new-privileges`
- **Shared Redis storage** for certificates (avoids rate-limit issues and credential sprawl)

## Recommended hardening

1. Use DNS-01 challenges in multi-instance setups (see `docs/dns-challenge.md`).
2. Set strong random values for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`.
3. Keep Redis on a private network only.
4. Uncomment `admin off` in the Caddyfile global options for production.
5. Start the WAF in DetectionOnly, review logs, then set `SecRuleEngine On`.
6. Restrict DNS API tokens to the minimum required permissions.
7. Use the health endpoint (`:8080`) for load balancer checks instead of exposing extra ports.
8. Keep containers updated and rebuild the custom Caddy image periodically.

## What this does not replace

- Network-level DDoS protection (use your cloud provider’s protections or a larger edge if needed)
- Full bot management / JavaScript challenges (possible future addition)
- Centralized log analysis (ship logs to your existing SIEM or Loki/Grafana)

## Production checklist

See `docs/production-checklist.md`.
