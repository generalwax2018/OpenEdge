# Production Checklist

Use this before putting OpenEdge in front of real traffic.

## Sovereignty & control

- [ ] Stack runs entirely in your Azure / AWS / private cloud account
- [ ] Redis is on a private network only (not publicly reachable)
- [ ] CrowdSec LAPI is on a private network (or properly firewalled)
- [ ] Logs are retained in your environment or shipped to your SIEM
- [ ] Strong, unique values set for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`

## Reverse proxy & HTTPS

- [ ] `DOMAIN` and `ACME_EMAIL` set correctly
- [ ] Domain points at the load balancer (or single node)
- [ ] Ports 80 and 443 open and forwarded
- [ ] DNS-01 enabled if running multi-instance or using wildcards (recommended)
- [ ] `admin off` uncommented in the Caddyfile for production

## WAF, rate limiting & threat intel

- [ ] WAF starts in DetectionOnly; false positives reviewed before switching to `On`
- [ ] Rate-limit zones reviewed for your traffic patterns
- [ ] CrowdSec bouncer key registered and working
- [ ] Common attack paths are blocked (default Caddyfile does this)
- [ ] Request body limits appropriate for your upload needs

## Multi-instance

- [ ] Shared Redis reachable by all nodes on private network
- [ ] Identical `REDIS_ADDR`, `REDIS_PASSWORD`, and Caddyfile on every node
- [ ] Load balancer health checks configured (port 8080 or TCP 80/443)
- [ ] DNS-01 preferred over HTTP-01 behind the load balancer
- [ ] Process documented for adding/removing nodes

## Operational

- [ ] Resource limits or autoscaling configured on nodes
- [ ] Monitoring/alerting on node health and 5xx rates
- [ ] Redis persistence or backup strategy decided (re-issuance works but consumes rate limits)
- [ ] Update process for the custom Caddy image documented

## Optional

- [ ] Central CrowdSec LAPI for multi-node consistency
- [ ] Anubis bot challenge enabled if bot traffic is high
- [ ] Log shipping to central system
