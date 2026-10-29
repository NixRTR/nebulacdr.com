---
title: ncclient Usage
linkTitle: Usage
weight: 20
---

After [installing ncclient](/docs/usage/ncclient/installation/), follow the page for the client you installed. Each one covers enrolling, day-to-day control, subnet routes and exit nodes, split-horizon DNS, re-enrolling, and troubleshooting.

| Client | Installed via | Page |
|--------|---------------|------|
| ncclient CLI | [Binaries](/docs/usage/ncclient/installation/binaries/) or [Pip](/docs/usage/ncclient/installation/pip/) | [ncclient CLI](/docs/usage/ncclient/usage/cli/) |
| Docker | [Docker image](/docs/usage/ncclient/installation/docker/) | [Docker](/docs/usage/ncclient/usage/docker/) |
| NixOS module | [`services.ncclient`](/docs/usage/ncclient/installation/nixos/) | [NixOS](/docs/usage/ncclient/usage/nixos/) |
| Linux app (and the headless Linux service) | [`.deb`, `.rpm`, or Flatpak](/docs/usage/ncclient/installation/linux/) | [Linux App](/docs/usage/ncclient/usage/linux/) |
| Windows app | [MSI installer](/docs/usage/ncclient/installation/windows/) | [Windows App](/docs/usage/ncclient/usage/windows/) |

## Concepts shared by every client

### Enrollment

A device joins Nebula Commander by **enrolling** with a one-time code:

1. In Nebula Commander, open **Nodes**, select the node for this device, and click **Enroll**.
2. Copy the code. It works once and expires if unused.
3. Give the code to the client (how depends on the client, see its page).

Enrolling stores a **device token** and the node's ID on the device. From v0.8.0 a device can enroll in several networks, one code each; see [Several networks on one device](/docs/usage/ncclient/usage/networks/). From then on the client uses the token to fetch the node's Nebula config and certificates, and reports a heartbeat so the node shows as online.

### Re-enrolling

Re-enroll when a device's token was lost or revoked, when you want the device to become a different node, or when moving it to another Nebula Commander server. Generate a **new** code (the old one is already used up) and enroll again.

Enrolling a node again **invalidates that node's previous token** on the server. A running client notices on its next poll (it gets a `401`), then picks up the new token from disk without a restart. From v0.8.0, a code for the same node on the same server replaces that network's token, while a code from a **different server** (or a different node) adds a network next to the old one; remove the old one afterwards. The per-client pages spell out the exact steps, and a couple of clients (Docker and the NixOS module) need an extra step first.

### When the server moves

From v0.9.0, when an admin moves Nebula Commander to a new address, devices follow
on their own after a waiting period (at least 1 hour, or 24 hours to another domain
or an IP) and a few checks, with no re-enrollment. The device shows "Moving to ..."
meanwhile. See [Move server](/docs/web-ui/move/#what-devices-do), including how to
limit this on a device with `move_policy`.

### Subnet routes, exit nodes, and DNS

- A [subnet router or exit node](/docs/usage/unsafe-routes/) assigned to a node is only **offered** to it. Each device has to accept it before using it.
- [Split-horizon DNS](/docs/web-ui/dns/) is served by lighthouses (usually the Docker client with `SERVE_DNS`). Other devices opt in to resolving through it.

Each client page shows how to do both.
