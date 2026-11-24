# OpenEdge

**Self-hosted edge for teams that want control and data sovereignty.**

OpenEdge gives small and medium businesses a production-ready reverse proxy, automatic HTTPS, WAF, rate limiting, threat intelligence, and bot protection that runs entirely in their own cloud account. Traffic never leaves their infrastructure for inspection.

It deploys as a single node or as multiple instances behind a load balancer, with shared state for certificates and rate limits.

## Why OpenEdge

| Pillar | What you get |
|--------|--------------|
| **Control & data sovereignty** | Everything runs in your Azure, AWS, or private cloud. No third-party edge sees your traffic. |
| **Reverse proxy + automatic HTTPS** | Caddy with automatic Let's Encrypt (HTTP-01 or DNS-01). Modern TLS defaults. |
| **WAF + rate limiting + threat intel** | Coraza + OWASP CRS, multi-zone rate limits (shared via Redis), CrowdSec IP reputation. |
| **Bot protection** | Known-bot blocking + optional Anubis proof-of-work challenge. |
| **Multi-instance in your cloud** | Shared Redis for certificates and rate limits. Load-balancer ready. Health checks included. |

## Architecture

```
Internet
   │
   ▼
Your Load Balancer (optional)
   │
   ├── OpenEdge node (Caddy + WAF + CrowdSec + rate limits + bot filters)
   ├── OpenEdge node
   └── ...
         │
         ▼
   Optional Anubis bot challenge
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

With optional bot challenge:

```bash
# In .env set UPSTREAM=http://anubis:3000 and ANUBIS_TARGET=http://your-origin:80
docker compose --profile bot-challenge up -d --build
```

Point your domain at the instance (or load balancer). Certificates are issued automatically.

## Documentation

- [Core capabilities](docs/capabilities.md)
- [Bot protection](docs/bot-protection.md)
- [Production checklist](docs/production-checklist.md)
- [Multi-instance](docs/multi-instance.md)
- [Azure](docs/deploy-azure.md)
- [AWS](docs/deploy-aws.md)
- [DNS-01 challenges](docs/dns-challenge.md)
- [Security](docs/security.md)
- [Security principles](docs/security-principles.md)

## Features

| Feature | Status |
|---------|--------|
| Automatic HTTPS (HTTP-01 & DNS-01) | Yes |
| Reverse proxy | Yes |
| WAF (OWASP CRS) with body limits | Yes |
| Multi-zone rate limiting (Redis-backed) | Yes |
| Threat intelligence (CrowdSec) | Yes |
| Known-bot User-Agent blocking | Yes |
| Bot challenge (Anubis) | Optional |
| Multi-instance + shared certificates | Yes |
| Security headers & attack path blocking | Yes |
| Health endpoint for load balancers | Yes |

## License

MIT
