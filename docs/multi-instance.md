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

**Requirements for shared certificates:**

1. Every node has the same `REDIS_ADDR` and `REDIS_PASSWORD`.
2. The load balancer forwards port 80 (HTTP-01 challenge) to the nodes, **or** you switch to DNS-01 challenges.
3. All nodes use the same `ACME_EMAIL` and domain configuration.

HTTP-01 works out of the box as long as the load balancer can reach any healthy node on port 80. DNS-01 is more reliable if you want to avoid depending on the load balancer for challenges.

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

## Azure example

1. Create an Azure Cache for Redis (or a small Redis VM) in the same VNet.
2. Create 2+ Linux VMs (or Container Apps / AKS pods) with Docker.
3. Deploy OpenEdge on each VM, pointing `REDIS_ADDR` at the shared Redis.
4. Put an Azure Load Balancer or Application Gateway in front, forwarding 80/443 to the VMs.
5. Point your domain at the load balancer public IP.
6. Health check: use port 8080 (`/`) or TCP on 80/443.

## AWS example

1. Create an ElastiCache Redis cluster (or a Redis EC2) in the private subnet.
2. Launch 2+ EC2 instances (or ECS/EKS tasks) with Docker.
3. Deploy OpenEdge on each, with `REDIS_ADDR` set to the ElastiCache endpoint.
4. Create an Application Load Balancer targeting the instances on 80/443.
5. Point the domain at the ALB.
6. Health check: use the optional port 8080 endpoint or TCP checks.

## Local multi-instance test

```bash
docker compose -f docker-compose.multi.yml up -d --build
```

## Production checklist

- [ ] Shared Redis with password and private networking only
- [ ] All nodes use the same `REDIS_ADDR` and `REDIS_PASSWORD`
- [ ] Load balancer forwards port 80 and 443
- [ ] Strong `CROWDSEC_BOUNCER_KEY`
- [ ] WAF starts in DetectionOnly; switch to On after tuning
- [ ] Redis is not publicly exposed
- [ ] Regular backups of Redis (or accept re-issuance on loss)
- [ ] Health checks configured on the load balancer
- [ ] Logging aggregated (or at least retained on each node)
- [ ] CrowdSec either central or running on each node

## Notes

- Certificates live in Redis. Losing Redis means nodes will re-issue certificates (watch Let's Encrypt rate limits).
- CrowdSec can be central (one LAPI) or local on each node.
- The optional `:8080` endpoint returns `ok` for simple health checks.
