---
title: Several networks on one device
linkTitle: Several networks
weight: 55
description: "From v0.8.0, one ncclient can be in several networks at once, each on its own Nebula Commander server if needed."
---

From v0.8.0, one device can be in several networks at once, for example your own network plus a customer's. The networks can be on the same Nebula Commander server or on different ones. You enroll once per network, with a code from that network's Nebula Commander, and the one ncclient on the device runs them all.

Each network gets its own Nebula process on its own interface, with its own token, config, certificates, routes and DNS. The desktop apps list them on a **Networks** page.

## Add a network

Enrolling again with a code from **another node** adds a network. Enrolling with a new code for the **same node on the same server** replaces that network's token instead, which is how you [re-enroll](/docs/usage/ncclient/usage/#re-enrolling).

| Client | How |
|--------|-----|
| [Windows app](/docs/usage/ncclient/usage/windows/) | **Add network** in the side bar (as administrator): server URL and code. |
| [Linux app](/docs/usage/ncclient/usage/linux/) | The **Add network** tab, or the **Add network** button on the Networks tab. |
| [ncclient CLI](/docs/usage/ncclient/usage/cli/) and the headless Linux service | `sudo ncclient enroll --server URL --code CODE`, once per network. |
| [NixOS module](/docs/usage/ncclient/usage/nixos/) | One entry per network in `services.ncclient.networks`. |
| [Docker](/docs/usage/ncclient/usage/docker/) | One network per container. Run a second container for a second network. |

A running service picks up a new network within a few seconds; nothing needs restarting.

```bash
sudo ncclient enroll --server https://nc.example.com --code XXXXXXXX       # first network
sudo ncclient enroll --server https://nc.customer.example --code YYYYYYYY  # another one
```

## List and leave networks

```bash
sudo ncclient networks list               # every network, with its interface, IP and state
sudo ncclient networks remove customer    # leave one, by name or id
```

`networks list` prints each network's id, name, interface, this node's IP in it, state and server. `networks remove` stops that network's Nebula and deletes its token and files. In the desktop apps, open the network and click **Leave network**.

Leaving a network on the device doesn't delete the node in Nebula Commander. Delete or revoke it there if the device shouldn't come back.

## Commands for one network

`ncclient routes` works on one network. With more than one enrolled, say which with `--network NAME` (the name or the id from `networks list`), **before** the subcommand:

```bash
sudo ncclient routes --network customer list
sudo ncclient routes --network customer accept 10.20.0.0/16
```

With only one network, leave `--network` out. In the desktop apps, routes, the exit node and the network's status are on that network's page.

Settings such as the poll interval and split-horizon DNS apply to every network on the device.

## Things to know

- **One exit node per device.** All of a device's internet traffic can only leave through one exit node. Accepting an exit node in one network replaces the one in use in another network. Subnet routes from different networks can be used at the same time, as long as they don't overlap.
- **Listen ports.** Two networks on one device can't bind the same UDP port. If they both ask for the same one (by default all nodes use `4242`), the second network gets a port the operating system picks, and peers learn it through the lighthouses. A lighthouse or relay needs a fixed port, so if one of the networks makes this device a lighthouse or relay, give the node in the other network a different **Listen port** in Nebula Commander. See [Listen port](/docs/usage/reachable-addresses/#listen-port).
- **Interface names.** Each network needs its own interface. Nebula Commander gives every network an interface name starting with `nc-` (shown and editable on the [network's page](/docs/web-ui/networks/#interface-name)). Networks on servers older than v0.8.0 all use `nebula1`; when a second such network joins a device, ncclient names its interface `nc-<network id>` instead, and the first one keeps `nebula1`. The name sticks, so it doesn't change back when the other network is removed.
- **Several ncclients on one Linux host.** If you run separate ncclient instances instead (for example several Docker containers on one host), every one must be v0.8.0 or newer. Older versions remove NetworkManager settings the others rely on and take their tunnels down. Only one instance can use the `/etc/resolv.conf` DNS fallback.

## Upgrading from before v0.8.0

On its first start, v0.8.0 or newer converts an existing single-network install to the new layout: the token moves to a per-network location and the network's files move into `networks/<id>/` under the output directory. Nothing else changes and the device stays enrolled.

This is one-way. Going back to a version before v0.8.0 afterwards needs a new enrollment.
