# OpenEdge – Self-Hosted Cloudflare-style Edge

**OpenEdge** is an open-source, self-hosted edge stack for small and medium businesses that want Cloudflare-like capabilities (reverse proxy, automatic HTTPS, WAF, rate limiting, threat intelligence, bot challenges) running in their own cloud account.

It works as a single node or as multiple instances behind a load balancer with shared Redis for certificates and rate-limit state.

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
   Optional Anubis bot challenge
         │
         ▼
   Shared Redis (certificates + rate limits)
         │
         ▼
   Your applications
```

## Quick Start

```bash
git clone https://github.com/generalwax2018/OpenEdge.git
cd OpenEdge
cp .env.example .env
# Edit .env
docker compose up -d --build
```

With optional bot challenge:

```bash
docker compose --profile bot-challenge up -d --build
# Set UPSTREAM=http://anubis:3000 in .env
```

## Documentation

- [Azure](docs/deploy-azure.md)
- [AWS](docs/deploy-aws.md)
- [Multi-instance](docs/multi-instance.md)
- [DNS-01 challenges](docs/dns-challenge.md)
- [Security features](docs/security.md)
- [Production checklist](docs/production-checklist.md)

## Features

| Feature | Status |
|---------|--------|
| Automatic HTTPS | Yes |
| DNS-01 challenges | Yes |
| Reverse proxy | Yes |
| WAF (OWASP CRS) | Yes |
| Threat intel (CrowdSec) | Yes |
| Multi-zone rate limiting | Yes |
| Bot challenge (Anubis) | Optional |
| Multi-instance + shared certs | Yes |
| Security headers | Yes |
| Health endpoint | Yes |

## License

MIT
