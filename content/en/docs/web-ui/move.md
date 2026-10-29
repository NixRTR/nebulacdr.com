---
title: Move server
linkTitle: Move server
weight: 58
description: "Move Nebula Commander to a new address (another domain, your own server, or an IP) and have every device follow on its own."
---

From v0.9.0, a system admin can move a Nebula Commander instance to a new
address, and enrolled devices **switch to the new server by themselves**, with
no re-enrollment and no editing of each device. Typical moves:

- `https://nebulacdr.net` to `https://beardedtek.nebulacdr.net`
- [Nebula Commander Cloud](/docs/cloud/) to your own domain, your own server, or
  an IP address ([leaving Cloud](#leaving-nebula-commander-cloud))
- an old server to new hardware with a new hostname

Both servers run side by side for a while: the old one tells devices where to go,
and they move over during that time.

## Before you start

- **Two-factor login on your account.** Starting or cancelling a move asks you to
  sign in again and enter a code, so a stolen password alone can't move your
  devices somewhere else. See [Security](/docs/web-ui/security/).
- **The new server**, installed and reachable at its new address
  ([Docker](/docs/installation/docker/) or [NixOS](/docs/installation/nixos/)),
  with nothing in it yet.
- **Devices on ncclient 0.9.0 or newer**, each checked in at least once after
  updating. The Move page shows which devices are **Ready**. A device on an older
  version, or one that hasn't checked in since updating, has to be pointed at the
  new server by hand (see [Backup & export](/docs/web-ui/backup/#keeping-devices-working)).

## Step by step

1. **Copy your data to the new server.** On the old server, open
   **Backup & export**, create an export with **Keep enrolled devices working
   after a move** ticked, then import it on the new server. See
   [Backup & export](/docs/web-ui/backup/).
2. **Copy the new server's fingerprint.** On the new server, open
   **Move server** (sidebar, system admins only) and click the copy button next
   to **This server's fingerprint**. It looks like `SHA256:M+NGf+0IigvT...`.
3. **Start the move on the old server.** Open **Move server** there and fill in:
   - **New server address**, for example `https://nc.example.com`
   - **New server's fingerprint**, pasted from step 2
   - **Devices wait before switching (hours)**: 24 by default, up to 72. This
     is time for anyone to notice and cancel a move they didn't expect.
   - **Keep this address answering for (days)**: 30 by default. See
     [After the move](#after-the-move).

   Click **Start move**, sign in again and enter your 2FA code. The old server
   checks that the fingerprint matches the server actually running at the new
   address, so a typo is caught right away.
4. **Watch devices move.** The old server shows a progress table: each device
   goes from **Waiting** to **Moved**. A device with a **Problem** shows why,
   and keeps trying.

While a move is in progress the old server is **read-only**: it shows a banner,
and changes to networks, nodes, users and so on are refused, so nothing is lost
between the export and the switch. Make further changes on the new server.

To call the move off, click **Cancel move** (sign-in and code again). Devices
that haven't switched yet stay where they are, and the old server is editable
again.

## What devices do

When a device sees the move, it shows a notice: "Moving to nc.example.com" in
`ncclient networks list` and in the Windows and Linux apps (the Linux app also
sends a desktop notification). Then it:

1. **Checks the move really comes from its server.** At enrollment (or the first
   check-in after updating to 0.9.0) each device remembers its server's key. A
   move is only followed if that key signed it.
2. **Waits.** At least as long as you set, and never less than **1 hour** for a
   new address in the same domain (`nebulacdr.net` to `beardedtek.nebulacdr.net`)
   or **24 hours** for a different domain or an IP address, whatever the server
   says. Moving to a different domain is shown as "moving to a different
   provider".
3. **Checks the new server.** The new server must prove it holds the key whose
   fingerprint you pasted, and must recognise this device (same node, same
   network).
4. **Switches.** It reports "moved" to the old server and carries on with the
   new one. Nebula keeps running throughout.

If any check fails, the device stays on the old server and tries again later.

### Limiting moves on a device

Whoever administers a device can restrict moves in its `settings.json`
(`/var/lib/ncclient/settings.json` on a Linux service, `C:\ProgramData\nebula-commander\settings.json`
on Windows):

```json
{ "move_policy": "same-domain" }
```

- `"any"` (default): follow moves to any address.
- `"same-domain"`: only follow moves within the same domain.
- `"none"`: never move; the device has to be re-pointed by hand.

Only editing the file changes this; neither a server nor the apps can.

A NixOS or Docker setup that declares the server address keeps the old address
in its configuration after a move. That's harmless (the device uses the server it
moved to), but update it when convenient.

## After the move

Devices that were switched off during the move still look for the old address
when they come back. So keep the old address answering for a while: the Move
page shows the recommended date.

The simplest way is **moved-only mode**: on the old server's Move page, click
**Responder file** to download `move-responder.json`, then restart the old
backend with

```sh
NEBULA_COMMANDER_MOVED_ONLY_FILE=/path/to/move-responder.json
```

(NixOS: `services.nebula-commander.movedOnlyFile`). The backend then does nothing
except tell devices where the server went; it needs no database and no sign-in
provider.

When the recommended period is over, Nebula Commander emails the admins (if
[SMTP](/docs/configuration/environment/#smtp-optional) is set up) with a tiny
standalone responder, `move_responder.py`, Python 3 only:

```sh
python3 move_responder.py move-responder.json --port 8099
```

Send only `/api/device/` on the old domain to it, and use the rest of the domain
for something else:

```nginx
location /api/device/ { proxy_pass http://127.0.0.1:8099; }
```

```caddy
handle /api/device/* { reverse_proxy 127.0.0.1:8099 }
```

Devices accept the move for a year by default, so stragglers can still find
their way.

## Leaving Nebula Commander Cloud

Moving off [Cloud](/docs/cloud/) to your own server works the same way:

1. Install Nebula Commander on your own server, at your own domain or IP.
2. On your Cloud instance, export with **Keep enrolled devices working after a
   move** ticked, and import it on your server.
3. Copy your server's fingerprint and start the move on your Cloud instance.

Because it's a different domain, devices wait at least 24 hours before
switching. Keep your Cloud membership until the Move page shows every device has
moved.

## Good to know

- Only system admins see the Move page.
- A move needs the new address over `https://`, unless the old server is itself
  plain `http://`.
- Starting and cancelling a move are recorded in the audit log.
