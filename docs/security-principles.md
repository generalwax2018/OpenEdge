# Security Principles

OpenEdge is a public-facing edge. These are the fundamental principles it follows and the expectations for a production deployment.

## 1. Encryption

**In transit (client → edge)**
- TLS 1.2+ only (explicit).
- Automatic certificates via Let's Encrypt (HTTP-01 or DNS-01).
- HSTS enabled.

**In transit (edge → origin)**
- Prefer HTTPS upstreams (`https://your-origin`).
- Caddy validates origin certificates by default. Use `tls_insecure_skip_verify` only for controlled internal cases and document why.

**At rest**
- Certificates and rate-limit state live in Redis. Enable Redis AUTH (password) and run Redis on a private network.
- Prefer Redis with TLS if it is not on a fully isolated network.
- Logs should be stored on encrypted volumes or shipped to an encrypted log system.
- Treat Redis and log volumes as sensitive.

## 2. Zero Trust (for a public edge)

Zero Trust here means: no request is trusted simply because it reaches the edge.

- Every request is inspected (WAF) and rate-limited.
- Reputation signals (CrowdSec) can block known-bad sources.
- Origins must not be directly reachable from the internet. Only the edge (or load balancer) should talk to them.
- Prefer origin authentication (shared secret header, mTLS, or cloud private networking) so a bypassed edge cannot reach the origin.
- No implicit trust of internal networks. Redis and CrowdSec LAPI stay private and authenticated.

This is not full user-to-app Zero Trust (that is Zscaler/Cloudflare Access territory). It is Zero Trust for public web traffic.

## 3. Least privilege

- Containers run as non-root with `no-new-privileges`.
- Caddy admin API is off by default.
- DNS provider tokens should have the minimum permissions required.
- CrowdSec bouncer keys are unique and rotatable.
- Redis is not exposed publicly and uses a password.

## 4. Defense in depth

Multiple independent controls:

1. Network (load balancer + private Redis/CrowdSec)
2. TLS termination
3. Attack path blocking
4. Rate limiting
5. WAF (OWASP CRS)
6. Threat intelligence (CrowdSec)
7. Optional bot challenge (Anubis)
8. Origin access controls

No single layer is assumed sufficient.

## 5. Secure defaults

- Admin API disabled.
- WAF starts in DetectionOnly (switch to On after tuning).
- Strong key generation is required and documented.
- Security headers are on by default.
- Request body and header size limits are set.

## 6. Auditability

- Access logs in JSON.
- WAF and CrowdSec decisions should be retained and reviewable.
- Certificate issuance and renewal events are logged by Caddy.

## What this does not cover

- Identity-aware access for users (add your own IdP / Cloudflare Access / Zscaler if needed).
- Full DLP.
- Global anycast DDoS absorption.
- Device posture checks.

See [security.md](security.md) for implementation details and [production-checklist.md](production-checklist.md) for go-live items.
