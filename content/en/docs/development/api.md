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
| `GET` | `/api/auth/dev-token` | No | **Development only.** When `DEBUG=true` or OIDC is not configured, returns a JWT (`token`, `expires_in`). Grants full admin access. Returns 404 in production when OIDC is configured. |
| `GET` | `/api/auth/me` | Optional | Current user info. Returns `{"authenticated": false}` or `{"authenticated": true, "sub", "email", "role", "system_role"}`. |
| `GET` | `/api/auth/login` | No | Redirects to the OIDC provider for login. Returns 501 if OIDC is not configured. |
| `GET` | `/api/auth/oidc-status` | No | OIDC provider readiness. Returns `{"status": "ok"}`, `{"status": "disabled"}`, or 503 if provider is unavailable. |
| `GET` | `/api/auth/callback` | No | OAuth callback. Exchanges authorization code for tokens and redirects to frontend with JWT in query (`/auth/callback?token=...`). |
| `GET` | `/api/auth/logout` | No | Logs out and redirects to OIDC logout (or frontend if OIDC not configured). |
| `POST` | `/api/auth/reauth/challenge` | Yes | Creates a reauthentication challenge for critical operations. Body: none. Response: `challenge`, `reauth_url`. Used before destructive actions (e.g. delete network). |
| `GET` | `/api/auth/reauth/callback` | No | Reauth OAuth callback. Validates state (challenge) and redirects to frontend with reauth token. |

---

## Heartbeat (`/api/nodes`)

Used by ncclient (or other clients) to report node liveness.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/nodes/{node_id}/heartbeat` | Yes (JWT) | Updates `last_seen` and sets node `status` to `active`. Call periodically from enrolled nodes. Response: `{"ok": true, "last_seen": "<iso>"}`. |

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
| `GET` | `/api/nodes` | Yes | List nodes. Query: optional `network_id`. Returns list of node objects (id, network_id, hostname, ip_address, groups, is_lighthouse, is_relay, status, etc.). |
| `GET` | `/api/nodes/{node_id}` | Yes | Get a single node by ID. |
| `PATCH` | `/api/nodes/{node_id}` | Yes | Update node. Body (all optional): `group`, `is_lighthouse`, `is_relay`, `public_endpoint`, `lighthouse_options`, `logging_options`, `punchy_options`. 409 if removing the only lighthouse. Response: `{"ok": true}`. |
| `DELETE` | `/api/nodes/{node_id}` | Yes | Delete node: release IP, remove host cert/key files, delete related records. 204. 409 if node is the only lighthouse. |
| `GET` | `/api/nodes/{node_id}/config` | Yes | Generate and return Nebula YAML config for the node (with inline PKI when key is stored). Response: `application/yaml` attachment. |
| `GET` | `/api/nodes/{node_id}/certs` | Yes | Return a ZIP with `ca.crt`, `host.crt`, optional `host.key`, and `README.txt`. |
| `POST` | `/api/nodes/{node_id}/revoke-certificate` | Yes | Revoke the node's certificate; node record is kept. Releases IP and removes cert/key files. Node can re-enroll later. Response: `{"ok": true}`. |
| `POST` | `/api/nodes/{node_id}/re-enroll` | Yes | Revoke existing cert (if any) and issue a new one for this node. Frontend typically creates an enrollment code afterward. Response: `{"ok": true, "node_id": id}`. |

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

---

## Users (`/api/users`)

System admin user management. All endpoints require **system-admin** role.

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `GET` | `/api/users` | System admin | List all users. Response: list of `{id, oidc_sub, email, system_role, created_at, network_count}`. |
| `GET` | `/api/users/{user_id}` | System admin | Get user details including `networks` (list of network id, name, role, permission flags). |
| `PATCH` | `/api/users/{user_id}` | System admin | Update user. Body: optional `system_role` (`system-admin` or `user`). |
| `DELETE` | `/api/users/{user_id}` | System admin | Delete user; removes all their network permissions. 204. |

---

## Node requests (`/api/node-requests`)

Request/approve workflow for node creation (e.g. when auto-approve is off).

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/node-requests` | Yes | Create a node request. Body: `network_id`, `hostname`, optional `groups`, `is_lighthouse`, `is_relay`. If user has manage_nodes or network has auto_approve_nodes, request is approved immediately and node is created. Response includes `status`, `created_node_id` if approved. |
| `GET` | `/api/node-requests` | Yes | List node requests. Query: optional `network_id`, `status`. Users see own requests; network owners/admins see requests for their networks; system admins see all. |
| `POST` | `/api/node-requests/{request_id}/approve` | Yes | Approve a pending request; creates the node and allocates IP. Body: empty object. Requires manage_nodes on the network. |
| `POST` | `/api/node-requests/{request_id}/reject` | Yes | Reject a pending request. Body: `reason`. Requires manage_nodes on the network. |

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
- **`/api/auth`** — Login, callback, dev-token, logout, reauth
- **`/api/nodes`** — Heartbeat, and full node CRUD + config/certs/revoke/re-enroll
- **`/api/networks`** — Networks CRUD, group firewall, check-ip, and network users (permissions)
- **`/api/certificates`** — Sign host cert (client key), create host cert (server key), list certs
- **`/api/device`** — Enrollment codes, enroll (public), config and certs (device token)
- **`/api/users`** — User list/detail/update/delete (system admin)
- **`/api/node-requests`** — Create/list/approve/reject node requests
- **`/api/access-grants`** — Create/list/revoke temporary admin access
- **`/api/invitations`** — Create/list/accept/resend/revoke invitations
- **`/api/audit`** — List audit log (system admin)
