# Multi-Instance Deployment

OpenEdge is designed to run as multiple instances behind a load balancer in your own Azure, AWS, or private cloud. Certificates and rate-limit state are shared via Redis so the cluster behaves as one edge.

## Architecture

```
Internet
   │
   ▼
Your Load Balancer (Azure LB / App Gateway / AWS ALB)
   │
   ├── OpenEdge node 1
   ├── OpenEdge node 2
   └── OpenEdge node N
         │
         ▼
   Shared Redis (private network)
   • certificates
   • rate-limit state
         │
         ▼
   Your origin applications
```

All inspection stays inside your account. Redis and CrowdSec remain on private networking.

## Certificate management

All nodes use the same Redis storage backend:

- Shared Let's Encrypt certificates (no duplicate issuance)
- Coordinated ACME challenges and renewals
- Any healthy node can serve the certificate
- Reduced risk of hitting Let's Encrypt rate limits

**Recommended:** Enable DNS-01 (see [dns-challenge.md](dns-challenge.md)). This removes dependence on the load balancer forwarding port 80 and supports wildcards.

Supported DNS providers: Cloudflare, AWS Route 53, Azure DNS.

## Rate limiting

Rate limits are distributed via Redis and apply across the whole cluster (eventually consistent, ~5s sync).

## Environment (set on every node)

```
ACME_EMAIL=admin@example.com
DOMAIN=example.com
UPSTREAM=http://your-origin:80
REDIS_ADDR=your-redis.private:6379
REDIS_PASSWORD=strong-password
CROWDSEC_BOUNCER_KEY=strong-random-key
CROWDSEC_LAPI_URL=http://central-crowdsec.private:8080   # or local
```

## Azure

1. Create Azure Cache for Redis (or a small Redis VM) in the same VNet.
2. Deploy 2+ nodes (VMs, Container Apps, or AKS) with Docker and the OpenEdge stack.
3. Point every node at the shared Redis.
4. Enable DNS-01 with Azure DNS (recommended).
5. Place an Azure Load Balancer or Application Gateway in front (80/443 → nodes).
6. Point the domain at the load balancer.
7. Health check: port 8080 or TCP 80/443.

## AWS

1. Create ElastiCache Redis (or Redis EC2) in a private subnet.
2. Launch 2+ nodes (EC2, ECS, or EKS) with the OpenEdge stack.
3. Point every node at the shared Redis.
4. Enable DNS-01 with Route 53 (recommended).
5. Create an Application Load Balancer targeting the nodes on 80/443.
6. Point the domain at the ALB.
7. Health check: port 8080 or TCP checks.

## Local multi-instance test

```bash
docker compose -f docker-compose.multi.yml up -d --build
```

## Notes

- Certificates live in Redis. Losing Redis causes re-issuance (watch Let's Encrypt rate limits).
- CrowdSec can be local on each node or central (preferred for consistency).
- See [production-checklist.md](production-checklist.md) before go-live.
