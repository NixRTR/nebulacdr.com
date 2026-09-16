---
title: API
linkTitle: API
weight: 40
---

The Nebula Commander backend exposes a REST API under the base path **`/api`**. All routes are prefixed with `/api`. Most endpoints require a valid JWT in the `Authorization: Bearer <token>` header unless noted otherwise.

## OpenAPI docs

When the backend is running in debug mode, interactive documentation is available at:

- **Swagger UI** – /api/docs
- **ReDoc** – /api/redoc

For full request/response schemas and parameters, use the interactive docs.

---

## Health and root

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api` | No | Root response: `name`, `version`, `status`. |
| `GET` | `/api/health` | No | Health check. Returns `{"status": "healthy"}`. |

---

## Auth (`/api/auth`)

Authentication and session management. OIDC (e.g. Keycloak) is used when configured; otherwise a dev token is available in debug mode.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/auth/dev-token` | No | **Development only.** Returns a JWT (`token`, `expires_in`) granting full admin access. Requires `DEBUG=true` (loopback requests only) or `NEBULA_COMMANDER_STANDALONE_ADMIN_BOOTSTRAP=true` when no OIDC provider is configured — it no longer falls back to enabled-by-default for standalone deployments. Returns 404 when OIDC is configured. |
| `POST` | `/api/auth/exchange` | No | Trade a one-time exchange `code` (from the `/auth/callback` or reauth redirect) for the real JWT. Single-use, 60-second-lived. Response: `{"token": "..."}`. 400 if invalid, already used, or expired. |
| `GET` | `/api/auth/me` | Optional | Current user info. Returns `{"authenticated": false}` or `{"authenticated": true, "sub", "email", "role", "system_role"}`. |
| `GET` | `/api/auth/login` | No | Redirects to the OIDC provider for login. Returns 501 if OIDC is not configured. |
| `GET` | `/api/auth/oidc-status` | No | OIDC provider readiness. Returns `{"status": "ok"}`, `{"status": "disabled"}`, or 503 if provider is unavailable. |
| `GET` | `/api/auth/callback` | No | OAuth callback. Exchanges the authorization code for tokens, then redirects to the frontend with a one-time **exchange code**, not the JWT itself (`/auth/callback?code=...`) — this keeps the token out of the URL, browser history, referrer headers, and access logs. The frontend immediately calls `POST /api/auth/exchange` to trade it for the real token. |
| `GET` | `/api/auth/logout` | No | Logs out and redirects to OIDC logout (or frontend if OIDC not configured). |
| `POST` | `/api/auth/reauth/challenge` | Yes | Creates a reauthentication challenge for critical operations. Body: none. Response: `challenge`, `reauth_url`. Used before destructive actions (e.g. delete network/node, revoke certificate, delete user). |
| `GET` | `/api/auth/reauth/callback` | No | Reauth OAuth callback. Validates state (challenge) and redirects to the frontend with a one-time exchange code for the reauth token (same exchange pattern as login). |

---

## Heartbeat (`/api/nodes`)

Used by ncclient (or other clients) to report node liveness.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/nodes/{node_id}/heartbeat` | Device token | Updates `last_seen` and sets node `status` to `active`. Call periodically from enrolled nodes using `Authorization: Bearer <device_token>` — not a human JWT. Body (all optional): `interval_seconds` (the client's actual poll interval, clamped 10–3600s, used by the dashboard to detect an offline node relative to its real cadence), `peer_reachability` (`{node_id: reachable}` — only honored when the reporting node is itself a lighthouse, checked server-side and scoped to its own network, so a node can't report on or spoof nodes it has no relationship to; see [Device: lighthouse-peers](#device-apidevice) below). Response: `{"ok": true, "last_seen": "<iso>"}`. |

---

## Networks (`/api/networks`)

Create and manage Nebula networks. Permissions are enforced per network (owner, member, and capability flags).

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/networks` | Yes | List networks the user can access. Includes `role`, `can_manage_nodes`, `can_invite_users`, `can_manage_firewall` per network. System admins see all networks (with limited data). |
| `POST` | `/api/networks` | Yes | Create a network. Body: `name`, `subnet_cidr`. Creator becomes owner. Returns full network object. 409 if name exists. |
| `GET` | `/api/networks/{network_id}` | Yes | Get a single network. System admins need an access grant to see CA path. |
| `PATCH` | `/api/networks/{network_id}` | Yes | Update network (owner only). Body: optional fields (currently no network-level firewall; use group firewall). |
| `DELETE` | `/api/networks/{network_id}` | Yes | Delete network (owner only; system admins can delete any). Body: `reauth_token`, `confirmation` (must match network name). 204 on success. |
| `GET` | `/api/networks/{network_id}/group-firewall` | Yes | List per-group firewall configs. Requires `can_manage_firewall`. Response: list of `{group_name, inbound_rules}`. |
| `PUT` | `/api/networks/{network_id}/group-firewall/{group_name}` | Yes | Create or update inbound firewall rules for a group. Body: `inbound_rules` (each: `allowed_group`, `protocol` (any/tcp/udp/icmp), `port_range`, optional `description`). |
| `DELETE` | `/api/networks/{network_id}/group-firewall/{group_name}` | Yes | Remove group firewall config for that group. 204 on success. |
| `GET` | `/api/networks/{network_id}/check-ip` | Yes | Check if an IP is available in the network. Query: `ip`. Response: `{"available": true or false}`. 400 if IP not in subnet. |

---

## Nodes (`/api/nodes`)

Manage Nebula nodes (hosts) within networks. Used by the Web UI and for manual cert/config workflows.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/nodes` | Yes | List nodes. Query: optional `network_id`. Returns only nodes the user can access (networks they have permission for, or node-level access grants). Response: list of node objects (id, network_id, hostname, ip_address, groups, is_lighthouse, is_relay, platform, status, etc.). |
| `GET` | `/api/nodes/{node_id}` | Yes | Get a single node by ID. |
| `PATCH` | `/api/nodes/{node_id}` | Yes | Update node. Body (all optional): `group`, `is_lighthouse`, `is_relay`, `public_endpoint`, `lighthouse_options`, `logging_options`, `punchy_options`, `platform` (`desktop`/`ios`/`android` — 400 if setting a non-desktop platform while the node is a lighthouse or relay). 409 if removing the only lighthouse. A `group` change automatically re-signs the node's certificate in place (same IP and keypair — a Nebula cert bakes its group in at signing time) when the node already has one. Response: `{"ok": true, "cert_resigned": true or false}`. |
| `DELETE` | `/api/nodes/{node_id}` | Yes | Delete node: release IP, remove host cert/key files, delete related records. Body: `reauth_token`, `confirmation` (must match node hostname). 204 on success. 409 if node is the only lighthouse. |
| `GET` | `/api/nodes/{node_id}/config` | Yes | Generate and return Nebula YAML config for the node (with inline PKI when key is stored). Response: `application/yaml` attachment. |
| `GET` | `/api/nodes/{node_id}/certs` | Yes | Return a ZIP with `ca.crt`, `host.crt`, optional `host.key`, and `README.txt`. |
| `POST` | `/api/nodes/{node_id}/revoke-certificate` | Yes | Revoke the node's certificate and take it offline; node record is kept and can re-enroll later. Body: `reauth_token`, `confirmation` (must match node hostname). Releases IP and removes cert/key files. Response: `{"ok": true}`. |
| `POST` | `/api/nodes/{node_id}/re-enroll` | Yes | Revoke existing cert (if any) and issue a new one for this node. Frontend typically creates an enrollment code afterward. Response: `{"ok": true, "node_id": id}`. |
| `PUT` | `/api/nodes/{node_id}/subnet-router` | Yes | Consumer-side pick of which other node's advertised subnets this node should route through. Body: `router_node_id` (int or `null` to clear). Atomically adds this node to every matching route's consumers on the chosen gateway and removes it from every other gateway's matching routes — a node uses at most one subnet router at a time. 400 if the target routes through itself, isn't on the same network, or doesn't advertise a subnet. See [Subnet Routers and Exit Nodes](/docs/usage/unsafe-routes/). |
| `PUT` | `/api/nodes/{node_id}/exit-node` | Yes | Same as above, for the exit-route pair (`0.0.0.0/0` + `::/0`) instead of subnet routes. Body: `exit_node_id`. |

---

## Certificates (`/api/certificates`)

Create or sign host certificates. Used when creating nodes from the Web UI or when using client-generated keys (e.g. betterkeys).

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/certificates/sign` | Yes | Sign a host certificate (client sends public key). Body: `network_id`, `name`, `public_key`, optional `group`, `suggested_ip`, `duration_days`. Response: `ip_address`, `certificate` (PEM), optional `ca_certificate`. Creates or updates node record. |
| `POST` | `/api/certificates/create` | Yes | Create a host certificate (server generates keypair). Body: `network_id`, `name`, optional `group`, `suggested_ip`, `duration_days`, `is_lighthouse`, `is_relay`, `public_endpoint`, `lighthouse_options`, `punchy_options`. Response: `node_id`, `hostname`, `ip_address`, `certificate`, `private_key`, optional `ca_certificate`. First node in network must be lighthouse. 409 if node name exists or suggested IP is taken. |
| `GET` | `/api/certificates` | Yes | List issued certificates. Query: optional `network_id`. Response: list of `{id, node_id, node_name, network_id, network_name, ip_address, issued_at, expires_at, revoked_at}`. |

---

## Device (`/api/device`)

Used by **ncclient** for enrollment and for fetching config/certs with a device token. Flow: create enrollment code (admin) → device redeems code at `POST /enroll` → device uses returned token for `GET /config` and `GET /certs`.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/device/enrollment-codes` | Yes (JWT) | Create a one-time enrollment code for a node. Body: `node_id`, `expires_in_hours` (default 24). Response: `code`, `expires_at`, `node_id`, `hostname`. Node must already have a certificate. |
| `POST` | `/api/device/enroll` | No | **Public.** Redeem a one-time code. Body: `code`. Response: `device_token`, `node_id`, `hostname`. Rate limited (e.g. 5 attempts per 15 min per IP). 404 if code invalid/expired, 400 if already used or expired. |
| `GET` | `/api/device/config` | Device token | Return Nebula YAML config for the device (inline PKI). Header: `Authorization: Bearer <device_token>`. Optional `If-None-Match: <etag>` for 304 when unchanged. |
| `GET` | `/api/device/certs` | Device token | Return ZIP with `ca.crt`, `host.crt`, optional `host.key`, `README.txt` for the device. |
| `GET` | `/api/device/dns-client-config` | Device token | Split-horizon DNS config for the device: `domain` and `dns_servers` (lighthouse Nebula IPs). 404 if DNS is not enabled for the network. Used by ncclient with `--accept-dns`. |
| `GET` | `/api/device/lighthouse-peers` | Device token | Return the other nodes on this device's network (`node_id`, `ip_address`, `hostname`), for a lighthouse to ping and report reachability on via its heartbeat's `peer_reachability`. Non-lighthouse devices always get an empty list — keeps client logic trivial and avoids leaking network topology to devices that don't need it. |
| `GET` | `/api/device/dnsmasq.conf` | Device token | dnsmasq zone config for this device's network (for lighthouse/container clients), derived from the token. Returns `text/plain`. Optional `If-None-Match: <etag>` for 304. Rate limited per device. |

---

## Networks DNS (`/api/networks/{network_id}/dns`)

Per-network DNS configuration for split-horizon DNS (domain and aliases). **Network owners only.**

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/networks/{network_id}/dns` | Yes | Get DNS config: `domain`, `enabled`, `upstream_servers`. Default domain is network name when not set. |
| `PUT` | `/api/networks/{network_id}/dns` | Yes | Update DNS config. Body: optional `domain`, `enabled`, `upstream_servers`. |
| `GET` | `/api/networks/{network_id}/dns/aliases` | Yes | List DNS aliases (hostname → node). Response: list of `{id, alias, node_id, node_hostname}`. |
| `POST` | `/api/networks/{network_id}/dns/aliases` | Yes | Create alias. Body: `alias` (hostname label), `node_id`. 409 if alias exists. |
| `DELETE` | `/api/networks/{network_id}/dns/aliases/{alias_id}` | Yes | Delete alias. 204. |
| `GET` | `/api/networks/{network_id}/dns/dnsmasq.conf` | Device token | Device-facing dnsmasq zone config, network given explicitly in the path rather than derived from the token. 403 if the token's node isn't a member of `network_id`. Otherwise identical to `/api/device/dnsmasq.conf` above (same ETag/304 behavior). |

---

## Users: self-service (`/api/users/me`)

Per-account preferences, available to any authenticated user (not just admins). See [Appearance](/docs/web-ui/appearance/).

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/users/me/theme` | Yes | Get the current user's active color theme: their saved overrides merged over the built-in defaults, so the response is always a complete token set. |
| `PUT` | `/api/users/me/theme` | Yes | Merge the given tokens into the current user's active theme (partial update — omitted keys are left as-is). Body: `theme` (map of token key → `{light, dark}` hex pair). Each key must be a known token; each color must match `^#[0-9a-fA-F]{6}$`. 400 on an unknown key or invalid color. |
| `GET` | `/api/users/me/themes` | Yes | List the current user's saved theme presets. Response: list of `{id, name, tokens, created_at}` — full token sets included, not just names. |
| `POST` | `/api/users/me/themes` | Yes | Save a named preset. Body: `name`, `tokens` (same shape as `PUT /me/theme`'s `theme`). 400 if the name is empty, over 100 characters, or already used by one of this user's other saved themes. There's no separate "apply" endpoint — applying a saved theme is just `PUT /me/theme` with its `tokens`. |
| `DELETE` | `/api/users/me/themes/{theme_id}` | Yes | Delete a saved theme. Scoped to the requesting user — 404 (not 403) if the theme belongs to someone else. 204 on success. |

---

## Users (`/api/users`)

System admin user management. All endpoints below require **system-admin** role (self-service `/me` endpoints above do not).

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/users` | System admin | List all users. Response: list of `{id, oidc_sub, email, system_role, created_at, network_count}`. |
| `GET` | `/api/users/{user_id}` | System admin | Get user details including `networks` (list of network id, name, role, permission flags). |
| `PATCH` | `/api/users/{user_id}` | System admin | Update user. Body: optional `system_role` (`system-admin` or `user`). |
| `DELETE` | `/api/users/{user_id}` | System admin | Delete user; removes all their network permissions. 204. |

---

## Access grants (`/api/access-grants`)

Temporary access for system admins to a specific network or node (e.g. for support). Only network owners can create grants.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/access-grants` | Yes | Create an access grant. Body: `admin_user_id`, `resource_type` (network or node), `resource_id`, `duration_hours`, `reason`. Target user must be system-admin. |
| `GET` | `/api/access-grants` | Yes | List grants. Query: `active_only` (default true). Network owners see grants they created; system admins see grants for them. |
| `DELETE` | `/api/access-grants/{grant_id}` | Yes | Revoke a grant. Only the user who created the grant can revoke. 204. |

---

## Invitations (`/api/invitations`)

Invite users to join a network by email. Requires can_invite_users (or owner) on the network.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/invitations` | Yes | Create invitation. Body: `email`, `network_id`, `role` (owner/member), `can_manage_nodes`, `can_invite_users`, `can_manage_firewall`, `expires_in_days` (default 7). Sends email if SMTP configured. 400 if user already member or pending invitation exists. |
| `GET` | `/api/invitations` | Yes | List invitations. Query: optional `network_id`, `status_filter`. Owners see invitations for their networks; system admins see all. |
| `GET` | `/api/invitations/public/{token}` | No | **Public.** Get invitation details by token (no auth). Returns network name, inviter, role, permissions, status, expires_at. 410 if expired. |
| `POST` | `/api/invitations/{token}/accept` | Yes | Accept invitation; creates NetworkPermission for current user. Must be logged in. Email should match invitation (optional check). |
| `POST` | `/api/invitations/{invitation_id}/resend` | Yes | Resend invitation email. Pending only. |
| `DELETE` | `/api/invitations/{invitation_id}` | Yes | Revoke invitation (inviter or network owner). Sets status to revoked. 204. |

---

## Network permissions (`/api/networks`)

Manage which users have access to a network and their roles/permissions. Owner-only for list/update/remove; can_invite_users for add.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/networks/{network_id}/users` | Yes | List users with access to the network. Owner or system admin. Response: list of user_id, email, role, can_manage_*, invited_by_*, created_at. |
| `POST` | `/api/networks/{network_id}/users` | Yes | Add user to network. Body: `user_id`, `role` (owner/member), `can_manage_nodes`, `can_invite_users`, `can_manage_firewall`. Requires can_invite_users. 400 if already member. |
| `PATCH` | `/api/networks/{network_id}/users/{target_user_id}` | Yes | Update user's permissions. Body: optional `role`, `can_manage_nodes`, `can_invite_users`, `can_manage_firewall`. Owner only. Cannot demote the last owner. |
| `DELETE` | `/api/networks/{network_id}/users/{target_user_id}` | Yes | Remove user from network. Owner only. Cannot remove the last owner. 204. |

---

## Audit (`/api/audit`)

Read-only audit log. **System admins only.**

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/audit` | System admin | List audit log entries. Query: `limit` (default 50, max 200), `offset`, optional `action`, `resource_type`, `from_date`, `to_date`. Ordered by occurred_at descending. Response: list of id, occurred_at, action, actor_*, resource_type, resource_id, result, details, client_ip. |

---

## Summary

- **`/api`** — Root, health
- **`/api/auth`** — Login, callback, exchange, dev-token, logout, reauth
- **`/api/nodes`** — Heartbeat (device token, carries `peer_reachability`), full node CRUD + config/certs/revoke/re-enroll, and subnet-router/exit-node consumer selection
- **`/api/networks`** — Networks CRUD, group firewall, check-ip, network users (permissions), and per-network DNS (config, aliases, device-facing dnsmasq.conf)
- **`/api/certificates`** — Sign host cert (client key), create host cert (server key), list certs
- **`/api/device`** — Enrollment codes, enroll (public), config and certs (device token), lighthouse-peers, dns-client-config and dnsmasq.conf for split-horizon DNS
- **`/api/users/me`** — Self-service: active color theme (get/set) and saved theme presets (list/create/delete), any authenticated user
- **`/api/users`** — User list/detail/update/delete (system admin)
- **`/api/access-grants`** — Create/list/revoke temporary admin access
- **`/api/invitations`** — Create/list/accept/resend/revoke invitations
- **`/api/audit`** — List audit log (system admin)
