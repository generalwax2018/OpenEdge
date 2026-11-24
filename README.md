# OpenEdge – Self-Hosted Cloudflare-style Edge

**OpenEdge** is an open-source, self-hosted edge stack for small and medium businesses that want Cloudflare-like capabilities (reverse proxy, automatic HTTPS, WAF, rate limiting, threat intelligence) running in their own cloud account.

It works as a single node or as multiple instances behind a load balancer with shared Redis for rate-limit state.

## Architecture

```
Internet
   │
   ▼
Load Balancer (optional)
   │
   ├── Caddy + Coraza WAF + CrowdSec + Rate limiting
   ├── Caddy + Coraza WAF + CrowdSec + Rate limiting
   └── ...
         │
         ▼
   Shared Redis (rate-limit state)
         │
         ▼
   Your applications
```

## Quick Start (single or multi)

```bash
git clone https://github.com/generalwax2018/OpenEdge.git
cd OpenEdge
cp .env.example .env
# Edit .env
docker compose up -d --build
```

For multi-instance testing:

```bash
docker compose -f docker-compose.multi.yml up -d --build
```

## Cloud deployment

- [Azure](docs/deploy-azure.md)
- [AWS](docs/deploy-aws.md)
- [Multi-instance](docs/multi-instance.md)

## Features

| Feature | Status |
|---------|--------|
| Automatic HTTPS | Yes |
| Reverse proxy | Yes |
| WAF (OWASP CRS) | Yes |
| Threat intel (CrowdSec) | Yes |
| Rate limiting | Yes (shared via Redis) |
| Multi-instance | Yes |
| Security headers | Yes |

## License

MIT
