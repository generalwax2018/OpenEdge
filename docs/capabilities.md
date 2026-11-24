# Core Capabilities

OpenEdge is built around four pillars. This document describes what each one delivers out of the box and how it is implemented.

## 1. Control and data sovereignty

All inspection, certificate management, rate limiting, and logging happen inside your infrastructure.

- Traffic never leaves your cloud account for WAF or rate-limit decisions.
- Certificates are stored in your Redis (or local volume on single-node).
- Logs stay on your nodes (or your log pipeline).
- No vendor lock-in. The stack is open source and containerized.
- You control rule updates, rate-limit thresholds, and CrowdSec collections.

This is the primary reason to choose OpenEdge over a commercial edge.

## 2. Reverse proxy + automatic HTTPS

Caddy terminates TLS and reverse-proxies to your origin.

- Automatic Let's Encrypt certificates (HTTP-01 by default).
- DNS-01 support for Cloudflare, Route 53, and Azure DNS (recommended for multi-instance and wildcards).
- Shared certificate storage via Redis so all nodes serve the same certs.
- Explicit TLS 1.2+ minimum.
- Security headers (HSTS, CSP, nosniff, frame options, etc.).
- Header size limits.

## 3. WAF + rate limiting + threat intelligence

Three layers of application protection:

**WAF (Coraza + OWASP CRS)**
- Starts in DetectionOnly so you can tune before blocking.
- Request body limits to reduce memory exhaustion risk.
- Common attack/scanner path blocking.

**Rate limiting**
- Multi-zone (general + sensitive paths).
- Distributed and shared via Redis across all instances.
- Configurable thresholds.

**Threat intelligence (CrowdSec)**
- Local or central LAPI.
- Collaborative IP reputation and blocking.
- Collections for Caddy, HTTP CVEs, and appsec rules.

## 4. Multi-instance in your own cloud

Designed for Azure, AWS, or any environment with a load balancer and private networking.

- Shared Redis for certificates and rate-limit state.
- Health endpoint (`:8080`) for load-balancer checks.
- Identical configuration on every node.
- DNS-01 recommended so certificate issuance does not depend on port 80 forwarding.
- Works behind Azure Load Balancer / Application Gateway or AWS ALB.

See [multi-instance.md](multi-instance.md) for deployment patterns and [production-checklist.md](production-checklist.md) for the go-live list.
