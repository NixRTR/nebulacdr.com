---
title: "State of Nebula Commander Development (v0.4.0)"
linkTitle: "State of Development (v0.4.0)"
date: 2026-09-16
draft: false
description: "Exit nodes finally shipped, every color in the app is now user-configurable, and the whole UI finally looks like one product — a look at everything since v0.3.0."
---

It's been a little over two weeks since [the v0.3.0 update](/blog/2026/08/30/2026-08-30-state-of-development-v0-3-0/), where we listed **exit nodes** as the one big thing still missing and called out a still-inconsistent UI as ongoing work. Neither of those is true anymore. This one covers **v0.3.0 through v0.4.0**.

## What's new since v0.3.0

**Subnet routers and exit nodes are here**, built on Nebula's own `unsafe_routes`. Turn any node into a subnet router (advertise a LAN behind it, Tailscale-`--advertise-routes`-style) or an exit node (route all of another node's traffic, Tailscale-`--advertise-exit-node`-style). Routes are opt-in per consumer — advertising one never silently exposes it network-wide — and configurable from either side: a gateway's "Used by" picker, or, new in this release, a simple **Use Subnet Router**/**Use Exit Node** dropdown on any other node that wants to consume one, without opening the gateway's own settings at all. On Linux nodes running `ncclient`, the whole thing is automatic: local subnet discovery, IP forwarding, and nftables NAT rules, no host access needed. Everything else (Windows, macOS, Docker-deployed `ncclient`, bare `nebula`) gets the right config generated for it, with [a full doc](/docs/usage/unsafe-routes/) on the manual host-side steps. Getting the Nebula semantics right here took two fix-forward releases early on — `via` is mandatory on every `unsafe_routes` entry and the gateway's own certificate needs a `-subnets` claim, or `nebula` refuses to start at all.

**Every color in the app is now yours to change.** The new Appearance page lets each user customize buttons, node status colors, type/OS badges, the group access diagram, and — new this release — the page background and a shared "container" background used by the sidebar, navbar, and every card, independently for light and dark mode. Every picker shows the hex code next to the swatch. You can also name and save color combinations as presets and switch between them later, a small personal theme library that's private to your account. None of this is shared or admin-controlled — it's a per-user preference, like the dark-mode toggle it lives next to.

**The UI finally looks like one product.** Nodes, Networks, and Groups moved to a consistent square-card grid layout earlier in the cycle; this release brings the **Home** dashboard and the admin **Users** page in line with the same pattern, replacing a full-width card list and a plain table respectively. Users' three separate view/edit-role/delete dialogs also collapsed into one details panel with an inline edit toggle, matching how Nodes and Groups already worked. And on the Nodes page specifically, the fields you touch constantly (platform, group, lighthouse/relay, the new subnet-router/exit-node pickers) stay visible, while the ones you set once and forget (logging, punchy/NAT tuning, the gateway-side route configuration) are tucked under an **Advanced** disclosure.

**A few real bugs got found and fixed along the way.** Badges on a node's card could grow tall enough on a narrow phone screen to cover the hostname and IP underneath them — fixed by keeping that text pinned above everything else, regardless of how many badges wrap. Wrapping the app in a themeable component surfaced a subtler one: the library's own internal dark-mode tracking could drift out of sync with our toggle after a single light/dark switch and then silently override it on every later page load — fixed by keeping both in sync on every toggle, not just at first load. And a SQLite id-reuse edge case that could crash network creation after a network had ever been deleted outside the normal flow is cleaned up automatically on startup now.

**The About screen tells you what you're actually running.** Frontend and backend version are both shown there now — the backend version was already available over the API, the frontend now gets the same release tag baked in at build time.

## Current status

Nebula Commander's core — networks, nodes, certificate lifecycle, firewall groups, Magic DNS, subnet routers and exit nodes, per-account theming, audit logging, invitations, multi-user management — is mature and running in production, and the Web UI is now visually consistent across all of it. The device client story hasn't changed since v0.3.0: CLI, Windows service, Docker, NixOS, and first-class mobile support.

What's still open: **automatic host-level route/NAT setup for subnet routers and exit nodes on Windows and Docker-deployed nodes** (Linux nodes running `ncclient` already get this for free) remains manual, and client hardening continues as real-world deployments surface edge cases.

See [About: Status](/about/) for the full implemented/planned breakdown.

## How to get involved

- **Run it** — [Server Installation](/docs/installation/) (Docker or NixOS), then the [Web UI](/docs/web-ui/) to create your first network and node.
- **Contribute** — Code and issues live on [GitHub](https://github.com/NixRTR/nebula-commander). See [Development](/docs/development/) for local setup, CI, and the API reference.
- **Join us on Matrix** — the [Nebula Commander Matrix Space](https://matrix.to/#/#nebula-commander:matrix.org) for discussion, support, and development conversation.

Thanks for following along with Nebula Commander.
