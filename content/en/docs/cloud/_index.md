---
title: Nebula Commander Cloud
linkTitle: Nebula Commander Cloud
weight: 15
description: "Hosted Nebula Commander: your own private instance, without running a server."
---

[Nebula Commander Cloud](https://cloud.nebulacdr.net/) is Nebula Commander hosted for you. You
get a private instance at `your-name.nebulacdr.net` with its own database, certificate
authority and sign-in. Backups, HTTPS and updates are handled for you.

It's the same open-source Nebula Commander described in the rest of these docs. Everything under
[Web UI](/docs/web-ui/) and [Client Usage](/docs/usage/) applies unchanged.

## Hosted or self-hosted?

| | Nebula Commander Cloud | Self-hosted |
|---|---|---|
| Price | Monthly subscription ([pricing](https://cloud.nebulacdr.net/pricing)) | Free (MIT licensed) |
| Server, HTTPS, sign-in service | Run for you | You run them ([installation](/docs/installation/)) |
| Backups and updates | Nightly encrypted backups, tested updates | Up to you |
| Your devices and lighthouses | On your machines | On your machines |
| Your data | Export any time from [Backup & export](/docs/web-ui/backup/) | On your server |

Either way, **only the control plane is hosted**. Your devices, lighthouses and relays run on
your own machines, and Nebula traffic goes directly between your devices. It never passes
through Nebula Commander Cloud.

You can move between the two at any time with an encrypted
[export and import](/docs/web-ui/backup/).

## Plans

- **Hosted:** the standard monthly price.
- **Name your own price:** the same instance and service. You choose the monthly amount, from the
  minimum shown on the [pricing page](https://cloud.nebulacdr.net/pricing), with no maximum.
  Paying more helps fund Nebula Commander's development. You can change the amount at any time.

Both plans include unlimited networks, nodes and users.

## In this section

- **[Signing up](/docs/cloud/signing-up/)**: create your instance and sign in for the first time.
- **[Managing your membership](/docs/cloud/managing-membership/)**: billing, changing your price,
  payment problems, cancelling and your data.

## Getting help

- Chat with us and other customers in the
  [Nebula Commander Cloud support room on Matrix](https://matrix.to/#/#nebula-commander-cloud-support:matrix.org).
  It's a public room, so never post passwords, keys or enrollment codes there.
- Or email [support@nebulacdr.net](mailto:support@nebulacdr.net).
