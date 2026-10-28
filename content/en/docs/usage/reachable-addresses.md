---
title: Public Endpoints and Reachable Addresses
linkTitle: Public Endpoints & Reachable Addresses
weight: 27
description: "Tell the mesh where to find a node on the underlay: a public endpoint and additional endpoints every peer knows about, extra addresses the node reports to its lighthouses, and the UDP port it listens on."
---

Nebula finds most nodes on its own: each node reports the addresses of its local
interfaces to the lighthouses, and the lighthouses also see the address a node
connects to them from. Three optional per-node settings cover what that misses. They
reach other nodes by different paths, so they're separate fields. A fourth setting,
the [listen port](#listen-port), is the UDP port the node itself binds.

| | Public endpoint | Additional endpoints | Additional reachable addresses |
|---|---|---|---|
| Where to set it | [Node details panel](/docs/web-ui/nodes/#the-node-details-panel) | Node details panel, once a public endpoint is set | Node details → **Advanced** |
| Values | One `host:port` (hostname, IPv4, or `[IPv6]`) | Up to 8 `host:port`, added one at a time | Up to 8 `IP:port`, added one at a time |
| Nebula setting | Every peer's [`static_host_map`](https://nebula.defined.net/docs/config/static-host-map/) | Every peer's `static_host_map`, after the public endpoint | This node's own [`lighthouse.advertise_addrs`](https://nebula.defined.net/docs/config/lighthouse/#lighthouseadvertise_addrs) |
| How other nodes learn it | Written into their config | Written into their config | Reported to the lighthouses, which pass it on |
| Lighthouses and relays | Need one | Allowed | Not shown (Nebula ignores it on lighthouses) |
| Changing it restarts Nebula on | Every node on the network | Every node on the network | That node only |

They can be used together: the endpoints let peers reach a node without asking a
lighthouse first, and the additional reachable addresses fill in paths the
lighthouses can't see.

## Additional endpoints

From v0.8.0, a node with a public endpoint can have up to 8 more endpoints. Every
peer gets the public endpoint **and** the additional endpoints in its
`static_host_map`, and Nebula tries each of them. Use them for addresses only some
nodes can reach. Nodes that can't reach an address just skip it.

The typical case is a lighthouse or relay inside an office:

1. Forward UDP `4242` on the office router to the machine, and give it a DHCP
   reservation.
2. Set its **public endpoint** to the office's public address, for example
   `203.0.113.7:4242`. Nodes outside the office use this one.
3. Add its **LAN address** as an additional endpoint, for example
   `192.168.1.50:4242`. Machines in the office reach it directly, without the
   router having to send traffic back to its own public address (NAT loopback,
   which many routers don't do for UDP).

To add one, open the node, click **Edit**, type a `host:port` under **Additional
endpoints** (it appears once the public endpoint is filled in), and click **Add**.
Use the trash button to remove one, then **Save**. Through the API, the field is
`extra_endpoints` on `PATCH /api/nodes/{id}`, a list with one endpoint per item.

Rules:

- **Only alongside a public endpoint.** Clearing the public endpoint removes the
  additional endpoints in the same save.
- **Same format as the public endpoint:** hostname, IPv4 or `[IPv6]`, plus a port.
  Private addresses are fine. Loopback, `0.0.0.0`, multicast, reserved addresses and
  addresses inside the Nebula network are rejected.
- An entry that repeats the public endpoint or another entry is dropped.
- Changing the list updates every peer's config, so Nebula restarts once on every
  node in the network. No certificate is re-signed.

Use **additional reachable addresses** instead when the node isn't a lighthouse and
you only want the lighthouses to hand the address out. Peers then learn it when they
look the node up, and nothing changes in every other node's config.

## Additional reachable addresses

### When to add them

Add the addresses Nebula can't discover itself:

- **A port forward** on the router in front of the node, especially when the
  external port isn't `4242`. Add the router's public IP and the forwarded port.
- **A second uplink**, or a LAN address that peers on the same site should try
  first.

You don't need to add the node's own interface addresses or the address the
lighthouse already sees; Nebula reports those automatically.

### Adding and removing addresses

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

### Rules

- **IP addresses only.** Nebula resolves a hostname here once at startup and
  doesn't start if the lookup fails, so hostnames are rejected.
- **Port `0`** means the node's own [listen port](#listen-port).
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

## Listen port

Every node listens on UDP `4242` unless you change **Listen port (UDP)** in the node
details panel. The value becomes the node's `listen.port`. The field is shown for
desktop nodes only. Give nodes different ports when:

- **Several nodes sit behind one NAT.** With the same source port, the NAT can keep
  the port for only one of them and remaps the others unpredictably. With distinct
  ports, each node gets its own predictable mapping and its own port forward.
- **One device is in more than one network.** Each network runs its own Nebula on
  that device, and two can't bind the same port. See
  [Several networks on one device](/docs/usage/ncclient/usage/networks/).

Rules:

- `0` lets the operating system pick a free port at each start. Peers learn the port
  through the lighthouses. Lighthouses and relays can't use `0`: peers reach them at
  their public endpoint, which needs a fixed port.
- Changing the port doesn't update the public endpoint. For a lighthouse or relay,
  point the public endpoint at the new port, or at a port forward to it.
- Changing the port restarts Nebula on that node only. No certificate is re-signed.
- Through the API, the field is `listen_port` on `PATCH /api/nodes/{id}`.
