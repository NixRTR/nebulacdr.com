---
title: Web UI Usage
linkTitle: Web UI Usage
weight: 35
---

The Nebula Commander web UI is a React dashboard that talks to the backend API. You use it to manage networks, nodes, certificates, and users. Most list pages — Networks, Nodes, Groups, Users, and the Home dashboard — use the same square-card grid layout, so once one feels familiar the rest do too.

| Light | Dark |
|---|---|
| ![Home dashboard](/screenshots/home.png) | ![Home dashboard in dark mode](/screenshots/dark/home.png) |

## Logging in

- **With OIDC** – When OIDC is configured ([OIDC](/docs/configuration/oidc/)), open the app URL and you are redirected to the provider (e.g. Keycloak). After login, you are sent back to the UI with a session.
- **Without OIDC (development)** – When the backend has `DEBUG=true` and OIDC is not configured, the backend exposes a dev-token endpoint. The UI can log you in without a real IdP. Do not enable this in production.

## Home

The **Home** dashboard is the first thing you see after logging in. Before you've
created a network and enrolled a node, it shows a getting-started walkthrough;
after that, it becomes a card-grid overview of every network you can access — the
same layout as the [Networks](/docs/web-ui/networks/) page, so it doubles as a
quick jumping-off point.

## Main flows

- **[Networks](/docs/web-ui/networks/)** – Create and manage Nebula networks. As a network owner you control nodes, IP allocation, and firewall groups.
- **[Groups](/docs/web-ui/groups/)** – Define security groups and inbound firewall rules per group.
- **[DNS](/docs/web-ui/dns/)** – Configure split-horizon DNS per network (domain and hostname aliases). Used by ncclient with `--accept-dns` and by the Docker lighthouse client.
- **[Nodes](/docs/web-ui/nodes/)** – Add nodes to networks, assign IPs, create or sign certificates, generate enrollment codes for ncclient, and configure [subnet routers and exit nodes](/docs/usage/unsafe-routes/).
- **[Client Download](/docs/web-ui/client-download/)** – Download ncclient (CLI, Linux `.deb`/`.rpm`/Flatpak, Windows app/MSI) served from this server.
- **[Invitations](/docs/web-ui/invitations/)** – Invite users to networks with roles and permissions (when OIDC is enabled).
- **[Users](/docs/web-ui/users/)** – System admins manage every user account and their system role.
- **[Appearance](/docs/web-ui/appearance/)** – Customize your own color theme, light and dark mode.

Certificates are created or signed from the Nodes page (Create or Sign flow). For Sign flow, the server does not have the private key; place `host.key` on the device (e.g. in the ncclient output directory).

## Responsive / mobile

The whole web UI is responsive - the same React app adapts down to phone-width
screens rather than being a separate mobile build. The sidebar collapses to a
hamburger menu, card grids reflow to fewer columns, and wide tables (like
Invitations) scroll horizontally within their own container instead of the
whole page:

| Home | Nodes | Invitations |
|---|---|---|
| ![Home on mobile](/screenshots/mobile/home.png) | ![Nodes on mobile](/screenshots/mobile/nodes.png) | ![Invitations on mobile](/screenshots/mobile/invitations-pending.png) |

Dark mode works the same way on mobile:

| Home | Nodes | Invitations |
|---|---|---|
| ![Home on mobile, dark mode](/screenshots/dark/mobile/home.png) | ![Nodes on mobile, dark mode](/screenshots/dark/mobile/nodes.png) | ![Invitations on mobile, dark mode](/screenshots/dark/mobile/invitations-pending.png) |

## Access control

Access is role-based: **system-admin**, **network-owner**, and **user**. Permissions include network-level and node-level access, plus access grants for temporary elevated admin access. Critical actions (e.g. delete network, node, or user) require reauthentication. Details are in the [repository docker/README](https://github.com/NixRTR/nebula-commander/tree/main/docker#user-roles-and-permissions).
