# Bot Protection

OpenEdge provides layered bot protection suitable for a self-hosted edge. It will not match Cloudflare Bot Management or commercial bot products, but it meaningfully reduces scraper, AI crawler, and basic automation traffic.

## Layers

| Layer | What it does | Status |
|-------|--------------|--------|
| **Rate limiting** | Limits requests per IP (general + sensitive paths) | Always on |
| **CrowdSec** | Blocks known-bad IPs and collaborative threat signals | Always on |
| **Known-bot User-Agent blocking** | Blocks obvious scanner/scraper UAs | Always on (lightweight) |
| **Anubis proof-of-work challenge** | Requires a browser to solve a challenge before reaching the origin | Optional |
| **CrowdSec bot detection** | Browser challenge + fingerprint scoring | Future (Caddy bouncer support limited) |

## Lightweight layer (always on)

The Caddyfile blocks common scanner and scraper User-Agents early. This stops the noisiest automated clients with almost no false positives.

## Anubis (recommended for heavy bot traffic)

Anubis sits between Caddy and your origin. Suspicious clients must solve a lightweight proof-of-work challenge. Legitimate browsers pass once and receive a cookie; most scrapers and AI crawlers cannot complete it.

### Enable

```bash
# In .env
UPSTREAM=http://anubis:3000
ANUBIS_TARGET=http://your-real-origin:80   # or https://
ANUBIS_DIFFICULTY=4

# Start with the profile
docker compose --profile bot-challenge up -d --build
```

### Tuning

- `ANUBIS_DIFFICULTY`: higher = harder challenge (4 is a reasonable default).
- Allow known good bots (search engines, uptime checks) via Anubis policy if needed.
- Monitor origin load; Anubis should significantly reduce scraper traffic.

## What this does not do

- Advanced ML bot scoring (1–99) like Cloudflare Bot Management
- Behavioral analysis across millions of sites
- Automatic good-bot verification at commercial scale
- JavaScript challenges with device fingerprinting (CrowdSec bot detection is moving this direction, but Caddy support is still limited)

## Future

CrowdSec 1.8+ added AppSec bot detection (challenge + fingerprint). Once the Caddy bouncer fully supports it, OpenEdge can enable it as a stronger optional layer without adding another component.

For now, Anubis + rate limiting + CrowdSec + known-UA blocking is the practical self-hosted set.
