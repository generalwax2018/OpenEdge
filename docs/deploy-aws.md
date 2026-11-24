# Deploy OpenEdge on AWS (single node)

Recommended for small/medium businesses: one EC2 instance with Docker.

## EC2 + Docker (simplest)

1. Launch an Ubuntu 22.04/24.04 LTS instance.
   - Type: t3.small or larger (2 vCPU / 2–4 GB RAM).
   - Security group: allow 22 (SSH), 80, 443 from the internet.
   - Optional: attach an Elastic IP.
2. Point your domain’s A record at the Elastic IP (or instance public IP).
3. SSH in and install Docker:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu
# log out and back in
```

4. Clone and deploy:

```bash
git clone https://github.com/generalwax2018/OpenEdge.git
cd OpenEdge
cp .env.example .env
nano .env   # set ACME_EMAIL, DOMAIN, strong CROWDSEC_BOUNCER_KEY
docker compose up -d --build
```

5. Check logs: `docker compose logs -f caddy`

Caddy obtains the Let’s Encrypt certificate automatically.

## Production tips

- Use an Elastic IP so the address doesn’t change.
- Consider an Application Load Balancer later if you add more nodes.
- Enable CloudWatch agent or simple health checks.
- Snapshot the EBS volume periodically.
- Keep the WAF in DetectionOnly until you’ve tuned false positives.
