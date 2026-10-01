---
title: Public Endpoints and Reachable Addresses
linkTitle: Public Endpoints & Reachable Addresses
weight: 27
description: "Tell the mesh where to find a node on the underlay: one public endpoint every peer knows about, plus extra addresses the node reports to its lighthouses."
---

Nebula finds most nodes on its own: each node reports the addresses of its local
interfaces to the lighthouses, and the lighthouses also see the address a node
connects to them from. Two optional per-node settings cover what that misses. They
reach other nodes by different paths, so they're separate fields.

| | Public endpoint | Additional reachable addresses |
|---|---|---|
| Where to set it | [Node details panel](/docs/web-ui/nodes/#the-node-details-panel) | Node details → **Advanced** |
| Values | One `host:port` (hostname, IPv4, or `[IPv6]`) | Up to 8 `IP:port`, added one at a time |
| Nebula setting | Every peer's [`static_host_map`](https://nebula.defined.net/docs/config/static-host-map/) | This node's own [`lighthouse.advertise_addrs`](https://nebula.defined.net/docs/config/lighthouse/#lighthouseadvertise_addrs) |
| How other nodes learn it | Written into their config | Reported to the lighthouses, which pass it on |
| Lighthouses and relays | Need one | Not shown (Nebula ignores it on lighthouses) |
| Changing it restarts Nebula on | Every node on the network | That node only |

Both can be used together: the public endpoint lets peers reach a node without
asking a lighthouse first, and the additional addresses fill in paths the
lighthouses can't see.

## When to add reachable addresses

Add the addresses Nebula can't discover itself:

- **A port forward** on the router in front of the node, especially when the
  external port isn't `4242`. Add the router's public IP and the forwarded port.
- **A second uplink**, or a LAN address that peers on the same site should try
  first.

You don't need to add the node's own interface addresses or the address the
lighthouse already sees; Nebula reports those automatically.

## Adding and removing addresses

1. Open the node, click **Edit**, and expand **Advanced**.
2. Under **Additional reachable addresses**, type one `IP:port`, for example
   `203.0.113.7:4242` or `[2001:db8::1]:4242`, and click **Add**. Repeat for each
   address.
3. Use the trash button next to an address to remove it.
4. Click **Save**.

The node picks up the change on its next config poll and restarts Nebula. The
certificate isn't re-signed, and no other node needs to restart.

Through the API, the field is `advertise_addrs` on `PATCH /api/nodes/{id}`: a list
with one address per item. Send `[]` to clear it. Leave the field out to keep the
current list.

## Rules

- **IP addresses only.** Nebula resolves a hostname here once at startup and
  doesn't start if the lookup fails, so hostnames are rejected.
- **Port `0`** means the node's own listen port (`4242`).
- **IPv6** goes in brackets: `[2001:db8::1]:4242`.
- **Rejected:** addresses inside the Nebula network itself, loopback, link-local,
  multicast, reserved space (`240.0.0.0/4`, including the broadcast address
  `255.255.255.255`), IPv4-mapped IPv6 (use the plain IPv4 address instead), and
  `0.0.0.0` / `::`.
- **At most 8** addresses per node. Duplicates are merged.
- **Addresses ending in `.0` or `.255`** are accepted but flagged with a warning:
  on a `/24` LAN they're the network or broadcast address and won't work, but on a
  larger LAN they can be ordinary hosts. Nebula Commander can't see your LAN's
  subnet mask, so it's up to you.

Additional reachable addresses were contributed by Austin Colt
([@YeeClaw](https://github.com/YeeClaw)).
