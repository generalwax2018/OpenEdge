# OpenEdge – Self-Hosted Cloudflare-style Edge (Single Node)

**OpenEdge** is an open-source, self-hosted edge stack that gives you Cloudflare-like capabilities (reverse proxy, automatic HTTPS, WAF, rate limiting, threat intelligence) that you can deploy on a single VPS, Azure VM, or AWS EC2 instance.

It is designed for **small and medium businesses**, homelabs, and teams that want full control and data sovereignty without depending on a third-party edge provider.

It is **not** a drop-in replacement for Cloudflare’s global anycast network and multi-terabit DDoS scrubbing. OpenEdge focuses on the practical features that matter for single-node deployments.

## Architecture (single node)

```
Internet
   │
   ▼
┌─────────────────────────────┐
│  Caddy (reverse proxy)      │  ← Automatic HTTPS (Let's Encrypt)
│  + Coraza WAF (OWASP CRS)   │  ← SQL injection, XSS, etc.
│  + CrowdSec bouncer         │  ← Known bad IPs / collaborative intel
│  + Rate limiting (Redis)    │  ← Per-IP abuse protection
└───────────┬──────────────────┘
             │
             ▼
       Your applications
```

## Quick Start (Docker Compose)

### Prerequisites
- Linux server (VPS, Azure VM, or AWS EC2) with Docker + Docker Compose
- A domain name pointed at the server’s public IP
- Ports 80 and 443 open

### Deploy

```bash
git clone https://github.com/generalwax2018/OpenEdge.git
cd OpenEdge
cp .env.example .env
# Edit .env — set ACME_EMAIL, DOMAIN, and a strong CROWDSEC_BOUNCER_KEY
docker compose up -d --build
docker compose logs -f
```

### First-time notes
- Caddy obtains Let’s Encrypt certificates automatically once the domain resolves.
- Coraza starts in **DetectionOnly** mode (logs attacks, does not block). Switch to `On` after tuning.
- Rate limiting is set to 100 requests/minute per IP by default. Edit `caddy/Caddyfile` to change it.
- CrowdSec needs a few minutes to populate decisions.

### Test the WAF / rate limit

```bash
# WAF test (should be logged or blocked depending on mode)
curl -I "https://yourdomain.com/?id=1' OR '1'='1"

# Rate limit test (run many times quickly)
for i in {1..120}; do curl -s -o /dev/null -w "%{http_code}\n" https://yourdomain.com/; done
```

## Cloud deployment guides

- [Azure (VM + Docker)](docs/deploy-azure.md)
- [AWS (EC2 + Docker)](docs/deploy-aws.md)

## Project Structure

```
openedge/
├── docker-compose.yml
├── .env.example
├── caddy/
│   ├── Caddyfile
│   └── Dockerfile
├── crowdsec/
│   └── acquis.yaml
├── docs/
│   ├── architecture.md
│   ├── deploy-azure.md
│   └── deploy-aws.md
└── README.md
```

## Roadmap (single-node first)

- [x] Automatic HTTPS + reverse proxy
- [x] WAF (Coraza + OWASP CRS)
- [x] Threat intelligence (CrowdSec)
- [x] Rate limiting (Redis-backed)
- [ ] Simple dashboard / status page
- [ ] Basic caching options
- [ ] Bot challenge integration (e.g. Anubis)
- [ ] Multi-node / GeoDNS (later)

## License

MIT

## Contributing

This is a practical starter for people who want to own their edge. Contributions welcome.
