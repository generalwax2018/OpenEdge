# Production Checklist

Use this before putting OpenEdge in front of real traffic.

## Encryption

- [ ] Client → edge uses TLS 1.2+ (default)
- [ ] Prefer HTTPS upstream (`https://` origin)
- [ ] Redis uses a password and is not publicly reachable
- [ ] Redis TLS enabled if not on a fully isolated network
- [ ] Logs stored on encrypted volumes or shipped to an encrypted system

## Zero Trust (edge)

- [ ] Origins are not directly reachable from the internet
- [ ] Origin firewall or private networking restricts access to the edge / load balancer only
- [ ] Optional: origin validates `X-OpenEdge-Secret` (or equivalent)
- [ ] Redis and CrowdSec LAPI require authentication and stay private
- [ ] No reliance on "internal network = trusted"

## Least privilege

- [ ] Caddy admin API remains off (default)
- [ ] Strong unique values for `CROWDSEC_BOUNCER_KEY` and `REDIS_PASSWORD`
- [ ] DNS provider tokens have minimum required permissions
- [ ] Containers run non-root with `no-new-privileges` (default)

## WAF, rate limiting & threat intel

- [ ] WAF starts in DetectionOnly; false positives reviewed before switching to `On`
- [ ] Rate-limit zones reviewed for your traffic
- [ ] CrowdSec bouncer key registered and working
- [ ] Request body limits appropriate for your uploads

## Multi-instance

- [ ] Shared Redis reachable by all nodes on private network
- [ ] Identical configuration and secrets on every node
- [ ] Load balancer health checks configured (port 8080 or TCP 80/443)
- [ ] DNS-01 preferred for multi-instance

## Operational

- [ ] Resource limits or autoscaling configured
- [ ] Monitoring/alerting on node health and 5xx rates
- [ ] Redis persistence or backup strategy decided
- [ ] Update process for the custom Caddy image documented

## Optional

- [ ] Central CrowdSec LAPI
- [ ] Anubis bot challenge
- [ ] Mutual TLS to origin
