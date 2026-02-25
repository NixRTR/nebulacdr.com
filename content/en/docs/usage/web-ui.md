---
title: Web UI
linkTitle: Web UI
weight: 10
---

The Nebula Commander web UI is a React dashboard that talks to the backend API. You use it to manage networks, nodes, certificates, and users.

## Logging in

- **With OIDC** – When OIDC is configured ([Configuration: OIDC](/docs/configuration/oidc/)), open the app URL and you are redirected to the provider (e.g. Keycloak). After login, you are sent back to the UI with a session.
- **Without OIDC (development)** – When the backend has `DEBUG=true` and OIDC is not configured, the backend exposes a dev-token endpoint. The UI can log you in without a real IdP. Do not enable this in production.

## Main flows

- **Networks** – Create and manage Nebula networks. As a network owner you control nodes, IP allocation, and firewall groups.
- **Nodes** – Add nodes to networks, assign IPs, create or sign certificates, and generate **enrollment codes** for ncclient.
- **Certificates** – Create CA and host certs from the UI (Create) or sign client-generated keys (Sign). For Sign flow, the server does not have the private key; place `host.key` on the device (e.g. in the ncclient output directory).
- **Users** – With OIDC, manage invitations and access. System admins can see all users; network owners invite users to networks or nodes and assign permissions. See [Configuration: OIDC](/docs/configuration/oidc/#roles-and-permissions) for roles.

## Access control

Access is role-based: **system-admin**, **network-owner**, and **user**. Permissions include network-level and node-level access, node request workflow, auto-approval, and access grants. Critical actions (e.g. delete network or node) require reauthentication. Details are in the [repository docker/README](https://github.com/NixRTR/nebula-commander/tree/main/docker#user-roles-and-permissions).
