# OpenEdge – Self-Hosted Cloudflare-style Edge

**OpenEdge** is an open-source, self-hosted edge stack that gives you Cloudflare-like capabilities (reverse proxy, automatic HTTPS, WAF, rate limiting, threat intelligence, and basic caching) that you can deploy on your own VPS or servers.

It is **not** a drop-in replacement for Cloudflare’s global anycast network and multi-terabit DDoS scrubbing. That requires dozens/hundreds of points of presence. OpenEdge focuses on the 80% of features that matter for most self-hosters, homelabs, small businesses, and security-conscious teams who want full control and data sovereignty.

## Architecture

```
Internet
   │
   ▼
┌─────────────────────────────┐
│  Caddy (reverse proxy)      │  ← Automatic HTTPS (Let's Encrypt)
│  + Coraza WAF (OWASP CRS)   │  ← SQL injection, XSS, etc.
│  + CrowdSec bouncer         │  ← Known bad IPs / collaborative intel
│  + Rate limiting            │
└───────────┬──────────────────┘
             │
             ▼
       Your applications
       (or another reverse proxy)
```

Optional future components:
- Multiple edge nodes + GeoDNS / anycast for true CDN
- Redis for shared caching / rate-limit state
- Simple control plane / dashboard
- AI-assisted bot detection and anomaly scoring (logs → local LLM or rules)

## Quick Start (Docker Compose)

### Prerequisites
- Linux VPS (or local machine for testing) with Docker + Docker Compose
- A domain name pointed at the server’s public IP (for real HTTPS)
- Ports 80 and 443 open

### Deploy

```bash
# Clone or copy this directory
cd openedge

# Copy and edit environment
cp .env.example .env
# Edit .env — set ACME_EMAIL at minimum

# Start the stack
docker compose up -d

# Watch logs
docker compose logs -f
```

### First-time notes
- Caddy will automatically obtain Let’s Encrypt certificates once the domain resolves to this server.
- Coraza starts in **DetectionOnly** mode by default (logs attacks but does not block). Switch to `On` after tuning false positives.
- CrowdSec needs a few minutes to populate decisions. You can force a test later.

### Test the WAF
Once running, try:

```bash
# Should be blocked (or logged in DetectionOnly)
curl -I "https://yourdomain.com/?id=1' OR '1'='1"
```

## Project Structure

```
openedge/
├── docker-compose.yml      # Full stack
├── .env.example
├── caddy/
│   ├── Caddyfile           # Reverse proxy + WAF + security config
│   └── Dockerfile          # Custom Caddy with Coraza + CrowdSec modules
├── crowdsec/
│   └── acquis.yaml         # Log acquisition config
├── docs/
│   └── architecture.md
└── README.md
```

## Roadmap / Ideas

- [ ] Multi-node edge + GeoDNS (PowerDNS or CoreDNS)
- [ ] Shared Redis cache + rate-limit state
- [ ] Simple web dashboard for rules, blocked IPs, and analytics
- [ ] AI bot / scraper detection (local model or CrowdSec + custom scenarios)
- [ ] Tunnel support (frp / rathole style) for origins behind NAT
- [ ] Plugin system for custom middleware
- [ ] Integration with existing tools (BunkerWeb, SafeLine, or Pingora-based edge)

## Existing Mature Alternatives

If you need something production-ready today rather than building:

| Project       | Stars  | Focus                          | Notes                          |
|---------------|--------|--------------------------------|--------------------------------|
| SafeLine      | ~23k   | WAF + reverse proxy            | Excellent detection, easy Docker |
| BunkerWeb     | ~11k   | WAF + full web server          | Nginx-based, great UI, plugins |
| NetGoat       | ~900   | Explicit CF alternative        | Early alpha, Go-based          |
| Coraza + Caddy| —      | Modern WAF engine              | Building block we use here     |

## License

MIT (or Apache-2.0 — choose what fits). All components used are permissively or copyleft licensed as appropriate.

## Contributing

This is a starter scaffold. Contributions welcome — especially around:
- Hardening the custom Caddy build
- Dashboard
- Multi-edge orchestration
- AI-powered detection modules

---

Built as a practical starting point for people who want to own their edge.
