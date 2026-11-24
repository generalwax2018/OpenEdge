# Multi-Instance Deployment (Azure / AWS)

OpenEdge supports multiple Caddy instances behind a load balancer. Rate-limit state is shared via Redis so limits apply across the whole cluster.

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
   Shared Redis (Azure Cache / ElastiCache / dedicated VM)
         │
         ▼
   Your origin applications
```

## Requirements

- Shared Redis reachable by all nodes (private network)
- Same `Caddyfile` and environment variables on every node
- Load balancer health checks on port 80 or 443
- Optional: one CrowdSec instance or CrowdSec on each node (they can share decisions via the LAPI if desired)

## Environment variables

Set these on every node:

```
ACME_EMAIL=admin@example.com
DOMAIN=example.com
UPSTREAM=http://your-origin:80
REDIS_ADDR=your-redis.example.com:6379
CROWDSEC_BOUNCER_KEY=strong-random-key
```

## Azure example

1. Create an Azure Cache for Redis (or a small Redis VM) in the same VNet.
2. Create 2+ Linux VMs (or Container Apps / AKS pods) with Docker.
3. Deploy OpenEdge on each VM, pointing `REDIS_ADDR` at the shared Redis.
4. Put an Azure Load Balancer or Application Gateway in front, forwarding 80/443 to the VMs.
5. Point your domain at the load balancer public IP.

## AWS example

1. Create an ElastiCache Redis cluster (or a Redis EC2) in the private subnet.
2. Launch 2+ EC2 instances (or ECS/EKS tasks) with Docker.
3. Deploy OpenEdge on each, with `REDIS_ADDR` set to the ElastiCache endpoint.
4. Create an Application Load Balancer targeting the instances on 80/443.
5. Point the domain at the ALB.

## Local multi-instance test

```bash
docker compose -f docker-compose.multi.yml up -d --build
```

This starts two Caddy instances sharing one Redis. Useful for testing distributed rate limits.

## Notes

- Rate limits are eventually consistent (default sync every 5 seconds).
- Certificates: each node can obtain its own Let’s Encrypt cert, or you can use a shared storage volume / certificate management.
- CrowdSec: you can run one central CrowdSec and point all bouncers at it, or run CrowdSec on each node.
- For production, do not expose Redis to the public internet.
