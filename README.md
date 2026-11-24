# OpenEdge

**Self-hosted edge for teams that want control and data sovereignty.**

OpenEdge gives small and medium businesses a production-ready reverse proxy, automatic HTTPS, WAF, rate limiting, and threat intelligence that runs entirely in their own cloud account. Traffic never leaves their infrastructure for inspection.

It deploys as a single node or as multiple instances behind a load balancer, with shared state for certificates and rate limits.

## Why OpenEdge

| Pillar | What you get |
|--------|--------------|
| **Control & data sovereignty** | Everything runs in your Azure, AWS, or private cloud. No third-party edge sees your traffic. |
| **Reverse proxy + automatic HTTPS** | Caddy with automatic Let's Encrypt (HTTP-01 or DNS-01). Modern TLS defaults. |
| **WAF + rate limiting + threat intel** | Coraza + OWASP CRS, multi-zone rate limits (shared via Redis), CrowdSec IP reputation. |
| **Multi-instance in your cloud** | Shared Redis for certificates and rate limits. Load-balancer ready. Health checks included. |

## Architecture

```
Internet
   │
   ▼
Your Load Balancer (optional)
   │
   ├── OpenEdge node (Caddy + WAF + CrowdSec + rate limits)
   ├── OpenEdge node
   └── ...
         │
         ▼
   Shared Redis (certificates + rate-limit state)
         │
         ▼
   Your applications (never leave your network for inspection)
```

## Quick Start

```bash
git clone https://github.com/generalwax2018/OpenEdge.git
cd OpenEdge
cp .env.example .env
# Edit .env — set DOMAIN, ACME_EMAIL, and strong keys
docker compose up -d --build
```

Point your domain at the instance (or load balancer). Certificates are issued automatically.

## Documentation

- [Core capabilities](docs/capabilities.md)
- [Production checklist](docs/production-checklist.md)
- [Multi-instance](docs/multi-instance.md)
- [Azure](docs/deploy-azure.md)
- [AWS](docs/deploy-aws.md)
- [DNS-01 challenges](docs/dns-challenge.md)
- [Security](docs/security.md)

## Features

| Feature | Status |
|---------|--------|
| Automatic HTTPS (HTTP-01 & DNS-01) | Yes |
| Reverse proxy | Yes |
| WAF (OWASP CRS) with body limits | Yes |
| Multi-zone rate limiting (Redis-backed) | Yes |
| Threat intelligence (CrowdSec) | Yes |
| Multi-instance + shared certificates | Yes |
| Security headers & attack path blocking | Yes |
| Health endpoint for load balancers | Yes |
| Optional bot challenge (Anubis) | Yes |

## License

MIT
