---
title: Reverse Proxy
linkTitle: Reverse Proxy
weight: 30
---

Nebula Commander is typically run **behind a reverse proxy** such as Nginx, Traefik, or Caddy.
The frontend and backend containers listen on plain HTTP inside Docker; your reverse proxy:

- Terminates TLS on port 443 with a certificate
- Proxies requests to the frontend container (and through it to the backend)
- Adds security headers like `Strict-Transport-Security` (HSTS)

This page shows recommended HSTS settings and example reverse proxy configurations.

## HSTS and HTTPS

For best security, configure HTTPS and HSTS on **your reverse proxy**, not inside the containers.
That lets you choose the right TLS policy for your environment (LAN-only, VPN-only, or Internet-facing).

Recommended baseline HSTS header:

```http
Strict-Transport-Security: max-age=31536000
```

For public Internet deployments on a DNS-backed domain, you can include subdomains:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Avoid `preload` and avoid HSTS on bare IPs or non-public hostnames:

- HSTS preload lists require a public, DNS-backed domain
- HSTS on an IP or \"internal-only\" hostname can cause problems if it is repurposed later

The examples below assume you already have TLS certificates (for example from Let’s Encrypt)
and that the Nebula Commander frontend container is reachable as `frontend:80` on a Docker network.

## Nginx example

Minimal Nginx configuration to put `nebula.example.com` behind HTTPS with HSTS:

```nginx
server {
    listen 443 ssl http2;
    server_name nebula.example.com;

    ssl_certificate     /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    # HSTS: adjust policy for your environment
    add_header Strict-Transport-Security "max-age=31536000" always;

    location / {
        proxy_pass http://frontend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Change:

- `nebula.example.com` to your actual hostname
- Certificate paths to where your certs are stored
- `frontend:80` if you renamed the frontend service or are not using Docker networking

## Traefik (file or dynamic configuration)

Traefik can apply HSTS via a headers middleware and route HTTPS traffic to the frontend service.
Example dynamic configuration (YAML) using the `websecure` entrypoint:

```yaml
http:
  middlewares:
    hsts-headers:
      headers:
        stsSeconds: 31536000
        stsIncludeSubdomains: true

  routers:
    nebula:
      rule: "Host(`nebula.example.com`)"
      entryPoints: ["websecure"]
      service: nebula-frontend
      middlewares:
        - hsts-headers
      tls:
        certResolver: letsencrypt

  services:
    nebula-frontend:
      loadBalancer:
        servers:
          - url: "http://frontend:80"
```

Adapt:

- `nebula.example.com` to your hostname
- `websecure` and `letsencrypt` to your Traefik entrypoint and certresolver names
- `frontend:80` to match your frontend container name and port

### Traefik with Docker labels

If you use the Nebula Commander Docker compose stack and Traefik’s Docker provider,
you can attach labels directly to the `frontend` service. Example:

```yaml
  frontend:
    image: ghcr.io/nixrtr/nebula-commander-frontend:latest
    container_name: nebula-commander-frontend
    restart: unless-stopped
    networks:
      - nebula-commander
      - traefik
    labels:
      - "traefik.enable=true"

      # Router: HTTPS entrypoint and host rule
      - "traefik.http.routers.nebula.rule=Host(`nebula.example.com`)"
      - "traefik.http.routers.nebula.entrypoints=websecure"
      - "traefik.http.routers.nebula.tls.certresolver=letsencrypt"

      # Service: forward to container port 80
      - "traefik.http.services.nebula-frontend.loadbalancer.server.port=80"

      # HSTS middleware
      - "traefik.http.middlewares.nebula-hsts.headers.stsSeconds=31536000"
      - "traefik.http.middlewares.nebula-hsts.headers.stsIncludeSubdomains=true"

      # Attach middleware to router
      - "traefik.http.routers.nebula.middlewares=nebula-hsts"
```

You must also ensure Traefik is on the same Docker network as the `frontend` service
(for example a shared `traefik` network).

Rename router (`nebula`), service (`nebula-frontend`), middleware (`nebula-hsts`),
entrypoint (`websecure`), and certresolver (`letsencrypt`) to fit your existing Traefik setup.

## Caddy example

Caddy can automatically obtain certificates and proxy to the frontend container:

```caddyfile
nebula.example.com {
    reverse_proxy frontend:80

    header {
        Strict-Transport-Security "max-age=31536000"
    }
}
```

Change `nebula.example.com` and `frontend:80` as needed. Caddy will handle TLS certificates
for you when properly configured with DNS and ports 80/443 exposed.
