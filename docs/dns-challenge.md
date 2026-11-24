# DNS-01 Challenge Support

DNS-01 is recommended for multi-instance deployments. It does not require the load balancer to forward port 80 for certificate issuance, and it supports wildcard certificates.

OpenEdge includes DNS provider modules for:

- **Cloudflare**
- **AWS Route 53**
- **Azure DNS**

## How to enable

1. Choose your provider and set the required environment variables (see below).
2. Uncomment the matching `tls` block in `caddy/Caddyfile`.
3. Rebuild and restart: `docker compose up -d --build`

## Cloudflare

Create an API token with:
- Zone → Zone → Read
- Zone → DNS → Edit

Environment:

```
CF_API_TOKEN=your_cloudflare_api_token
```

Uncomment in Caddyfile:

```
tls {
	dns cloudflare {$CF_API_TOKEN}
}
```

## AWS Route 53

Create an IAM user or role with Route 53 permissions for the hosted zone.

Environment:

```
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=us-east-1
```

Uncomment in Caddyfile:

```
tls {
	dns route53 {
		access_key_id {$AWS_ACCESS_KEY_ID}
		secret_access_key {$AWS_SECRET_ACCESS_KEY}
		region {$AWS_REGION:us-east-1}
	}
}
```

## Azure DNS

Create a service principal with DNS Zone Contributor on the resource group, or use a managed identity (omit client credentials).

Environment:

```
AZURE_SUBSCRIPTION_ID=...
AZURE_RESOURCE_GROUP_NAME=...
AZURE_TENANT_ID=...
AZURE_CLIENT_ID=...
AZURE_CLIENT_SECRET=...
```

Uncomment in Caddyfile:

```
tls {
	dns azure {
		subscription_id {$AZURE_SUBSCRIPTION_ID}
		resource_group_name {$AZURE_RESOURCE_GROUP_NAME}
		tenant_id {$AZURE_TENANT_ID}
		client_id {$AZURE_CLIENT_ID}
		client_secret {$AZURE_CLIENT_SECRET}
	}
}
```

## Wildcard certificates

With DNS-01 you can request wildcards. Change the site address to:

```
*.example.com, example.com {
	...
}
```

## Notes

- DNS-01 is more reliable than HTTP-01 behind load balancers.
- Keep API tokens restricted to the minimum required permissions.
- Certificates are still stored in the shared Redis, so all nodes can use them.
- HTTP-01 remains available if you do not enable DNS-01.
