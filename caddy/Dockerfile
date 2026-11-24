# Custom Caddy with Coraza WAF + CrowdSec bouncer
# Build with: docker compose build caddy

ARG CADDY_VERSION=2.10

FROM caddy:${CADDY_VERSION}-builder-alpine AS builder

RUN xcaddy build \
	--with github.com/corazawaf/coraza-caddy/v2 \
	--with github.com/hslatman/caddy-crowdsec-bouncer/http \
	--with github.com/hslatman/caddy-crowdsec-bouncer/appsec

FROM caddy:${CADDY_VERSION}-alpine

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

# Create log directory
RUN mkdir -p /var/log/caddy && chown -R caddy:caddy /var/log/caddy

# Default user is already non-root in official images
USER caddy

EXPOSE 80 443 443/udp

CMD ["caddy", "run", "--config", "/etc/caddy/Caddyfile", "--adapter", "caddyfile"]
