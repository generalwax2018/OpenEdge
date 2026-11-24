# Production Checklist for SMBs

Use this when deploying OpenEdge in your own Azure or AWS account.

## Must-have

- [ ] Domain pointed at the load balancer (or single node)
- [ ] Ports 80 and 443 open and forwarded
- [ ] Strong values for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`
- [ ] Redis not exposed to the public internet
- [ ] WAF in DetectionOnly mode until false positives are tuned
- [ ] Let's Encrypt email set (`ACME_EMAIL`)

## Multi-instance specific

- [ ] Shared Redis reachable by all nodes on private network
- [ ] Same `REDIS_ADDR` and `REDIS_PASSWORD` on every node
- [ ] Load balancer health checks configured (port 8080 or TCP 80/443)
- [ ] All nodes have identical Caddyfile and environment

## Recommended

- [ ] Central CrowdSec or CrowdSec on each node
- [ ] Log shipping (or at least log retention)
- [ ] Redis persistence or backups (optional — re-issuance works but burns rate limits)
- [ ] Resource limits / autoscaling on the nodes
- [ ] Monitoring / alerting on node health and 5xx rates
- [ ] Documented process for adding/removing nodes

## Optional later

- [ ] DNS-01 challenges (more reliable than HTTP-01 behind some load balancers)
- [ ] Central logging dashboard
- [ ] Bot challenge integration
- [ ] Simple status / admin UI
