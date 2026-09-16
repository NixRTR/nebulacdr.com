---
title: "State of Nebula Commander Development (v0.3.0)"
linkTitle: "State of Development (v0.3.0)"
date: 2026-08-30
draft: false
description: "Nebula Commander has grown up since our last update: mobile support, lighthouse-based reachability monitoring, Nebula v2 certs, and a NixOS module for the device client, not just the server."
---

It's been a little over six months since our [last state-of-development post](/blog/2026/02/24/2026-02-24-state-of-development/), when we described Nebula Commander as being in "early development" with a client that was "still evolving." Since then the project has moved through v0.2.12 all the way to **v0.3.0**, and the picture has changed enough that it's worth a proper update.

## What's new since February

**The client is no longer the weak link.** The Windows client was rearchitected around a real background service (`NebulaCommanderService`, running as `LocalSystem`) with a thin, always-unelevated tray app on top — no UAC prompts, ever, for enrolling, starting Nebula, or applying DNS. It ships three ways now: a standalone CLI, the Windows MSI (CLI + tray + service), and a Docker image with an s6-overlay supervisor for lighthouse deployments.

**Mobile is a first-class platform, not an afterthought.** iOS and Android nodes (via the official [Nebula app from defined.net](https://defined.net)) get their own `platform` field, their own status handling, and a dedicated config-download flow in the Web UI — no `ncclient` required, since there's nowhere to run it on a phone. Split-horizon DNS follows each platform's actual capabilities: automatic on iOS (`NEDNSSettings.matchDomains` is genuinely domain-scoped), an explicit opt-in on Android (whose `VpnService` DNS API has no such scoping).

**Certificates moved to Nebula v2**, with P256 available as an opt-in curve alongside the original Curve25519, and a real correctness fix: editing a node's group in the Web UI now automatically re-signs its certificate in place — no re-enrollment, no IP churn — since a Nebula cert bakes its group in at signing time and the two used to silently drift apart.

**You can now tell when a node actually went dark.** A lighthouse can ping the other nodes on its network and report reachability back on its own heartbeat; the server validates that report against the lighthouse's real network membership (so a node can't spoof status for peers it has no relationship to) and surfaces it on the Nodes dashboard alongside each node's own check-in state — genuine node-offline detection, not just "haven't heard from it in a while."

**Security got a dedicated pass.** All 56 open Dependabot alerts and all 64 code-scanning alerts (CodeQL + Bandit) are closed, including a real PowerShell-injection bug in the Windows DNS-apply code. The dev-token bootstrap endpoint no longer works by default on a standalone (no-IdP) deployment — you opt in explicitly now. The JWT secret fails closed the same way the encryption key always has: leave it at the placeholder default, and the server refuses to boot. Reauthentication (step-up auth) now covers node deletion and certificate revocation, not just network deletion.

**NixOS gets the client, not just the server.** `services.nebula-commander` picked up production-auth options it was missing (OIDC, `publicUrl`, CORS, session security), and there's a genuinely new `services.ncclient` module for running the device client declaratively — built, tested against a real NixOS system build, and documented in [Server Installation: NixOS](/docs/installation/nixos/).

## Current status

Nebula Commander is no longer "early development." The core control plane — networks, nodes, certificate lifecycle (including automatic re-signing), firewall groups, Magic DNS, audit logging, invitations, multi-user management — is mature and running in production. The Web UI covers all of it. The device client is no longer the caveat it was in February: CLI, Windows service, Docker, and now NixOS, plus first-class mobile support.

What's still open: **exit nodes** (full-tunnel routing via Nebula's `unsafe_routes`) remain unimplemented, and client hardening continues as real-world deployments surface edge cases — most recently a Docker persistence bug where a container recreation could silently lose a node's identity even though its device token survived, found and fixed by watching a real production lighthouse.

See [About: Status](/about/) for the full implemented/planned breakdown.

## How to get involved

- **Run it** — [Server Installation](/docs/installation/) (Docker or NixOS), then the [Web UI](/docs/web-ui/) to create your first network and node.
- **Contribute** — Code and issues live on [GitHub](https://github.com/NixRTR/nebula-commander). See [Development](/docs/development/) for local setup, CI, and the API reference.
- **Join us on Matrix** — the [Nebula Commander Matrix Space](https://matrix.to/#/#nebula-commander:matrix.org) for discussion, support, and development conversation.

Thanks for following along with Nebula Commander.
