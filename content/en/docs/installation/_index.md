---
title: Server Installation
linkTitle: Server Installation
weight: 20
---

You can run Nebula Commander in several ways:

| Method                                             | Best for                                                                |
|----------------------------------------------------|-------------------------------------------------------------------------|
| [Docker (recommended)](/docs/installation/docker/) | Quick start and production: pre-built images, `docker compose`         |
| [NixOS](/docs/installation/nixos/)                 | NixOS hosts: systemd module with options for port, database, certs, JWT |
| [Reverse Proxy](/docs/installation/reverse-proxy/) | Nginx, Traefik, or Caddy in front with TLS and HSTS                    |

Choose one and follow the linked page. For local development (venv, uvicorn, frontend dev server), see [Development: Setup](/docs/development/setup/). After installation, see [Server Configuration](/docs/configuration/) to set the database, JWT, and optional OIDC.
