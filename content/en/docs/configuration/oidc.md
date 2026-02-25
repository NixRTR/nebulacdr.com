---
title: OIDC Authentication
linkTitle: OIDC
weight: 20
---

Nebula Commander can use OpenID Connect (OIDC) for login. Keycloak is supported with a pre-configured setup; other OIDC providers (Authentik, Auth0, etc.) work with the same environment variables.

## Required variables

Set these in `env.d/backend` (Docker) or in the backend environment:

- **`NEBULA_COMMANDER_PUBLIC_URL`** – URL where users reach the app (FQDN or host:port). The redirect URI is derived as PUBLIC_URL + `/api/auth/callback` if you do not set `OIDC_REDIRECT_URI`.
- **`NEBULA_COMMANDER_OIDC_ISSUER_URL`** – Issuer URL used by the **backend** to talk to the provider. Inside Docker use the container name and container port (e.g. `http://keycloak:8080/realms/nebula-commander`). Do not use the host-mapped port here.
- **`NEBULA_COMMANDER_OIDC_PUBLIC_ISSUER_URL`** – Issuer URL as seen by the **browser** (FQDN or host:port). Use the same URL users would use to open the provider’s login page.
- **`NEBULA_COMMANDER_OIDC_CLIENT_ID`** – OIDC client ID (e.g. `nebula-commander`).
- **`NEBULA_COMMANDER_OIDC_CLIENT_SECRET`** – Client secret from the provider. Prefer `NEBULA_COMMANDER_OIDC_CLIENT_SECRET_FILE` in production.

Optional: **`NEBULA_COMMANDER_OIDC_REDIRECT_URI`** – Override only if you need a callback URL different from PUBLIC_URL + `/api/auth/callback`.

## Keycloak with Docker

1. Copy example env files and edit `env.d/backend` (set PUBLIC_URL, OIDC issuer URLs, client id/secret).
2. Start with Keycloak:  
   `docker compose -f docker-compose.yml -f docker-compose-keycloak.yml up -d`
3. Create a user in Keycloak Admin and assign client roles under the `nebula-commander` client: `system-admin`, `network-owner`, or `user`.

### Zero-touch Keycloak

The project provides a custom Keycloak image that imports the realm and theme at startup. Build the image (see [Installation: Docker](/docs/installation/docker/) and docker/keycloak), set the same OIDC variables in `env.d/backend`, and start the stack. No manual realm creation is required; create users and assign roles in the Admin console.

### Manual Keycloak setup

If you do not use the zero-touch image, create the realm and client in Keycloak:

- Client type: OpenID Connect, confidential, standard flow.
- Valid redirect URI: your PUBLIC_URL + `/api/auth/callback`.
- Valid post-logout redirect URIs: PUBLIC_URL + `/*`.
- Web origins: your PUBLIC_URL.
- Create client roles: `system-admin`, `network-owner`, `user`. Assign them to users via Role mapping.

Then set the client secret in `env.d/backend`.

## Backend cannot reach Keycloak

The backend talks to Keycloak on the Docker network. Use the **container** port in `NEBULA_COMMANDER_OIDC_ISSUER_URL` (e.g. `http://keycloak:8080/realms/nebula-commander`), not the host-mapped port. Use the host port or FQDN only in `NEBULA_COMMANDER_OIDC_PUBLIC_ISSUER_URL` and in `NEBULA_COMMANDER_PUBLIC_URL`.

## External OIDC provider

Use the same variables. In the provider, set redirect URI to PUBLIC_URL + `/api/auth/callback`, client type Confidential, grant type Authorization Code. Set `OIDC_ISSUER_URL` and `OIDC_PUBLIC_ISSUER_URL` to the provider’s issuer URL (often the same). Run without the Keycloak compose file.

## Roles and permissions

- **system-admin** – Manage users; view all networks/nodes with limited data; delete networks/nodes (with reauth); request access grants from owners.
- **network-owner** – Create and manage networks, nodes, invitations, node requests, and access; set default node parameters and auto-approval.
- **user** – Access only what network owners grant; request nodes and higher access.

See the [repository docker/README](https://github.com/NixRTR/nebula-commander/tree/main/docker) for detailed RBAC and permission features.
