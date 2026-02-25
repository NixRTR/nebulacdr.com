---
title: API
linkTitle: API
weight: 50
---

The Nebula Commander backend exposes a REST API under the base path **`/api`**. All routes are prefixed with `/api`.

## OpenAPI docs

When the backend is running, interactive documentation is available at:

- **Swagger UI** – [/api/docs](/api/docs)
- **ReDoc** – [/api/redoc](/api/redoc)

Use these to explore and try endpoints. Authentication (session or bearer token) is required for most routes.

## Routers

| Prefix | Tag | Purpose |
|--------|-----|---------|
| `/api/auth` | auth | Login, callback, dev-token, logout |
| `/api/nodes` | heartbeat | Node heartbeat (used by ncclient) |
| `/api/networks` | networks | Create and manage networks |
| `/api/nodes` | nodes | Manage nodes (CRUD, enroll) |
| `/api/certificates` | certificates | CA and host certificate operations |
| `/api/device` | device | Device client API (config, certs for ncclient) |
| `/api/users` | users | User management |
| `/api/node-requests` | node-requests | Node request workflow |
| `/api/access-grants` | access-grants | Temporary access grants |
| `/api/invitations` | invitations | Invitations and acceptance |
| `/api/networks` | network-permissions | Network-level permissions |

Health and root:

- `GET /api/health` returns `{"status": "healthy"}`.
- `GET /api` returns a simple root response.

For full request/response schemas and parameters, use the interactive docs at `/api/docs`.
