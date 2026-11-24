# Multi-Instance Deployment (Azure / AWS)

OpenEdge supports multiple Caddy instances behind a load balancer. Certificates and rate-limit state are shared via Redis.

## Architecture

```
Internet
   │
   ▼
Azure LB / AWS ALB
   │
   ├── OpenEdge node 1 (Caddy + optional CrowdSec)
   ├── OpenEdge node 2
   └── OpenEdge node N
         │
         ▼
   Shared Redis (certificates + rate limits)
         │
         ▼
   Your origin applications
```

## Certificate management

All nodes use the same Redis storage backend. This gives you:

- Shared Let's Encrypt certificates (no duplicate issuance)
- Coordinated ACME challenges and renewals (distributed locks)
- Automatic failover — any node can serve the certificate
- Protection against Let's Encrypt rate limits

### Recommended: DNS-01 challenges

For multi-instance deployments, enable DNS-01 (see [docs/dns-challenge.md](dns-challenge.md)). This avoids depending on the load balancer to forward port 80 for HTTP-01 challenges and supports wildcards.

Supported providers out of the box: Cloudflare, AWS Route 53, Azure DNS.

HTTP-01 still works if the load balancer forwards port 80 to healthy nodes.

## Rate limiting

Rate limits are distributed and eventually consistent (default 5-second sync). The limit applies across the whole cluster.

## Environment variables

Set these on every node:

```
ACME_EMAIL=admin@example.com
DOMAIN=example.com
UPSTREAM=http://your-origin:80
REDIS_ADDR=your-redis.example.com:6379
REDIS_PASSWORD=your-redis-password
CROWDSEC_BOUNCER_KEY=strong-random-key
```

Plus the DNS provider variables if using DNS-01.

## Azure example

1. Create an Azure Cache for Redis (or a small Redis VM) in the same VNet.
2. Create 2+ Linux VMs (or Container Apps / AKS pods) with Docker.
3. Deploy OpenEdge on each VM, pointing `REDIS_ADDR` at the shared Redis.
4. Enable DNS-01 with Azure DNS (recommended) or ensure the load balancer forwards port 80.
5. Put an Azure Load Balancer or Application Gateway in front, forwarding 80/443 to the VMs.
6. Point your domain at the load balancer public IP.
7. Health check: use port 8080 (`/`) or TCP on 80/443.

## AWS example

1. Create an ElastiCache Redis cluster (or a Redis EC2) in the private subnet.
2. Launch 2+ EC2 instances (or ECS/EKS tasks) with Docker.
3. Deploy OpenEdge on each, with `REDIS_ADDR` set to the ElastiCache endpoint.
4. Enable DNS-01 with Route 53 (recommended).
5. Create an Application Load Balancer targeting the instances on 80/443.
6. Point the domain at the ALB.
7. Health check: use the optional port 8080 endpoint or TCP checks.

## Local multi-instance test

```bash
docker compose -f docker-compose.multi.yml up -d --build
```

## Production checklist

See [production-checklist.md](production-checklist.md) and [security.md](security.md).

## Notes

- Certificates live in Redis. Losing Redis means nodes will re-issue certificates (watch Let's Encrypt rate limits).
- CrowdSec can be central (one LAPI) or local on each node.
- The optional `:8080` endpoint returns `ok` for simple health checks.
