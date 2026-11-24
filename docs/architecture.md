# OpenEdge Architecture Notes

## Why this stack?

- **Caddy**: Best-in-class automatic HTTPS, simple config, excellent performance, active community.
- **Coraza**: Modern, memory-safe (Go) WAF engine fully compatible with OWASP CRS and ModSecurity rules. Faster and lighter than classic ModSecurity in many benchmarks.
- **CrowdSec**: Collaborative threat intelligence. Your node contributes (anonymously) and benefits from decisions made across thousands of other deployments. Excellent for blocking known scanners, brute-forcers, and bad actors.
- **Docker Compose**: Lowest barrier to entry. One command to deploy.

## Limitations vs Cloudflare

| Feature                  | Cloudflare          | OpenEdge                          |
|--------------------------|---------------------|-----------------------------------|
| Global anycast network   | Yes (300+ PoPs)     | No (you provide the servers)      |
| Massive DDoS scrubbing   | Yes                 | Basic (rate limit + CrowdSec)     |
| Automatic HTTPS          | Yes                 | Yes                               |
| WAF                      | Yes                 | Yes (OWASP CRS + custom)          |
| Caching / CDN            | Yes                 | Basic (extend with Redis/Varnish) |
| Workers / edge compute   | Yes                 | Possible via Caddy plugins        |
| Data sovereignty         | No                  | Yes                               |
| Cost                     | Freemium / paid     | Your VPS cost                     |

## Scaling to a real CDN

To get closer to Cloudflare:

1. Run OpenEdge on multiple VPS in different regions (Hetzner, OVH, DigitalOcean, etc.).
2. Use GeoDNS (PowerDNS, CoreDNS + geoip, or a managed DNS with geo routing) to send users to the nearest edge.
3. Shared state: Redis or a small database for rate limits, cache, and blocked IPs.
4. Origin shielding: one or more “origin” nodes that only accept traffic from your edges.
5. Optional: anycast IP if you can obtain one (harder for individuals).

## Tuning the WAF

OWASP CRS has false positives. Recommended workflow:

1. Run with `SecRuleEngine DetectionOnly`.
2. Generate real traffic + known attack payloads.
3. Review Coraza logs.
4. Add exceptions for legitimate traffic.
5. Switch to `SecRuleEngine On`.

See official CRS documentation for exclusion rules.

## Security notes

- Keep the Caddy and CrowdSec containers updated.
- The custom Caddy build pulls modules at build time — pin versions in production.
- Do not expose the CrowdSec LAPI (port 8080) to the public internet.
- Use strong values for `CROWDSEC_BOUNCER_KEY`.
