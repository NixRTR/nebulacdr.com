---
title: Nodes
linkTitle: Nodes
weight: 30
---

The **Nodes** page shows every node across networks (or filtered to one network) as
a grid of square cards. You create nodes, assign IPs and groups, create or sign
certificates, generate **enrollment codes** for [ncclient](/docs/usage/ncclient/),
download config, and — new since v0.3.7 — pick which other node each one should
route through as a [subnet router or exit node](/docs/usage/unsafe-routes/).

![Nodes page](/screenshots/nodes.png)

## The node grid

Each card is color-coded by status — never checked in, active, or inactive — and
carries a bottom-right badge cluster: **Lighthouse**, **Relay**, and a type/OS badge
(**iOS**, **Android**, **Windows**, **Linux**, **macOS**, or a generic **Node**
fallback until the platform is known). Hostname and IP sit at the top of the card,
always drawn above the badges so a long badge row can never hide them, even on a
narrow phone screen. Click a card to open its details panel:

![Node details panel](/screenshots/nodes-detail.png)

All of these colors,
including the status backgrounds and badge colors, are user-configurable — see
[Appearance](/docs/web-ui/appearance/).

## Creating a node

1. Open **Nodes** and optionally filter by network.
2. Click **Create Node**. Pick a network first if you haven't filtered to one.
3. Enter a **hostname** (e.g. `laptop-alice`, `server-1`). The hostname identifies the node and is used in certificates and config.
4. The server assigns an **IP address** from the network's subnet, or you may suggest one.
5. Set the node's **group** (e.g. `laptops`, `servers`). The group is used for [firewall rules](/docs/web-ui/groups/) and must match a group defined for that network.
6. Set the **platform** — Desktop (runs `ncclient`), iOS, or Android.
7. Submit. The node is created, its certificate is issued, and its card appears in the grid.

## The node details panel

Click a card to open its details. The fields visible by default are the ones you're
likely to change often:

| Field | Description |
|-------|-------------|
| **Platform** | Desktop, iOS, or Android. Converting an existing node changes how it's enrolled — see [Mobile nodes](#mobile-nodes-iosandroid) below. |
| **Group** | Nebula security group for this node. Used for firewall (see [Groups](/docs/web-ui/groups/)). |
| **Lighthouse** | If enabled, this node acts as a Nebula lighthouse (others can punch through to it). Desktop only. |
| **Relay** | If enabled, this node can relay traffic for other nodes. Desktop only. |
| **Use Subnet Router** | Pick another node on this network to route this node's traffic to its advertised subnets through. See [Subnet Routers and Exit Nodes](/docs/usage/unsafe-routes/). |
| **Use Exit Node** | Pick another node to route *all* of this node's traffic through (full-tunnel). Same doc as above. |

### Advanced

Settings you set once and rarely touch again are tucked under an **Advanced**
disclosure at the bottom of the panel:

- **Logging** — Nebula's own log level, format, and timestamp options.
- **Punchy** — NAT hole-punching behavior (respond, delay, respond delay).
- **Subnet Router & Exit Node Config** — the *gateway* side of routing: which local
  subnets this node advertises (auto-discovered on Linux nodes running `ncclient`,
  or entered by hand under **Other**), plus a **"Used by"** picker per route
  controlling which other nodes may consume it.
- **Exit Node** — the gateway-side exit-node toggle (**Exit node (route all
  traffic)**) and its own "Used by" picker.

If you're setting up a node to *be* a subnet router or exit node for others, that's
all under Advanced. If you just want this node to *use* one that already exists,
the visible **Use Subnet Router**/**Use Exit Node** dropdowns are all you need.

## Certificates: Create vs Sign

For each node you need a host certificate. Two flows:

- **Create certificate** – The server generates the private key and certificate. You can download a bundle that includes `host.key`, `host.crt`, and `ca.crt`. Use this when the device does not already have a key (e.g. ncclient or you will copy the bundle to the device).
- **Sign certificate** – You generate the private key on the device; the server only signs the cert. The server never has `host.key`. After signing, download `host.crt` and `ca.crt` and place them on the device next to your existing `host.key`.

Choose Create for simplicity when the server can hold the key (or when you will deploy the bundle once). Choose Sign when you want the key to never leave the device.

## Enrollment code (for ncclient)

After the node exists and has a certificate, you can generate an **enrollment code** for [ncclient](/docs/usage/ncclient/). Open the node's card and click **Get Enrollment Code**. Copy the one-time code. On the device run:

```bash
ncclient enroll --server https://YOUR_NEBULA_COMMANDER_URL --code XXXXXXXX
```

The device stores a token and can then use `ncclient run` to pull config and certs. See [Client Download](/docs/web-ui/client-download/) for binaries.

## Download config

You can download the node's Nebula config and certs (e.g. `config.yaml`, `ca.crt`, `host.crt`, and `host.key` if Create was used). Use this for [manual (nebula) setup](/docs/usage/nebula/) or backup.

## Re-enroll and delete

- **Re-enroll** – If a device was enrolled but the token is lost or expired, generate a new enrollment code and run `ncclient enroll` again on the device.
- **Delete node** – Removes the node and its certificate from Nebula Commander. Requires reauthentication and typing the node's hostname to confirm. Devices using that node will need a new node or re-enrollment if you recreate it.

## Mobile nodes (iOS/Android)

There's no `ncclient` agent for mobile, so mobile nodes skip the enroll-code flow
entirely. Set a node's **platform** (Desktop, iOS, or Android) when creating or
editing it — a mobile node can't also be a lighthouse or relay. Instead of an
Enroll button, a mobile node's card shows a **download config.yaml** action: that
file is the same one `GET /nodes/{id}/config` always generates (inline
cert/key/CA already embedded), which is exactly the format the official Nebula
app from [defined.net](https://defined.net)'s "Add Site → From file" import
expects.

Split-horizon DNS for mobile is platform-specific: **iOS** applies it
automatically whenever the network has DNS enabled (no per-node opt-in needed);
**Android** has no scoped equivalent, so it's an explicit opt-in checkbox — when
enabled, *all* device DNS routes through the network's lighthouse(s) while
connected.
