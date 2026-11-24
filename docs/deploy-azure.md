# Deploy OpenEdge on Azure (single node)

Recommended for small/medium businesses: one Linux VM with Docker.

## Option A: Azure VM + Docker (simplest)

1. Create a Linux VM (Ubuntu 22.04/24.04 LTS recommended).
   - Size: B2s or larger (2 vCPU / 4 GB RAM minimum; 4 GB+ preferred).
   - Allow ports 22, 80, 443 in the Network Security Group.
2. Point your domain’s A record at the VM’s public IP.
3. SSH in and install Docker:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
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

Caddy will automatically obtain a Let’s Encrypt certificate once the domain resolves.

## Option B: Azure Container Apps / Container Instances

Possible but more involved (need persistent volumes for Caddy data and CrowdSec). The VM approach is recommended for most SMBs.

## Production tips

- Use a managed domain and DNS.
- Enable Azure Backup or snapshot the VM.
- Set a strong `CROWDSEC_BOUNCER_KEY`.
- Start WAF in DetectionOnly, review logs, then switch to On.
- Monitor with Azure Monitor or simple uptime checks.
