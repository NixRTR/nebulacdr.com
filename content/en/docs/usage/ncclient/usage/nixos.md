---
title: NixOS
linkTitle: NixOS
weight: 30
---

This page covers day-to-day use of the `services.ncclient` module. See [NixOS installation](/docs/usage/ncclient/installation/nixos/) for importing the module and its full option list. Paths below use the defaults (`stateDir = "/var/lib/ncclient"`, `outputDir = "/var/lib/ncclient/nebula"`); adjust them if you changed those options.

## What the module sets up

- `ncclient.service`: runs `ncclient run` as root, with `nebula` from `nebulaPackage`, plus `nftables` and `iproute2` (for subnet-router/exit-node forwarding and NAT), on its PATH. It restarts on failure.
- `ncclient-enroll.service`: only when `enrollCodeFile` or `networks` is set. A oneshot that runs `ncclient enroll` before the main service, **once per network**: after enrolling it writes a marker file (`/var/lib/ncclient/.enrolled-<name>`) and never sends that network's code again.
- The D-Bus policy and polkit rules the [Linux desktop app](/docs/usage/ncclient/usage/linux/) uses, if you also enable `services.ncclient-desktop`. Only members of `adminGroups` (default `wheel`/`sudo`) can make changes without a password.
- An `ncclient` command on the system PATH, pre-pointed at the service's state. See [Running ncclient commands](#running-ncclient-commands).

## Where state lives

| Path | What it is |
|------|------------|
| `/var/lib/ncclient/tokens/` | One device token per network |
| `/var/lib/ncclient/settings.json` | Each network's server, node ID and accepted routes |
| `/var/lib/ncclient/.enrolled-<name>` | Marker: the enroll unit has enrolled network `<name>` |
| `/var/lib/ncclient/nebula/networks/<id>/` | Each network's Nebula `config.yaml`, certificates, status |

Both directories are `0700 root:root`. Before v0.8.0 the token was `/var/lib/ncclient/token` and the Nebula files were directly in `nebula/`; ncclient moves them into this layout on its first start.

## Enroll

The declarative way is `enrollCodeFile`: a file holding a one-time code from **Nodes → Enroll**. For example, a sops-nix secret, or a root-only file you create by hand:

```nix
services.ncclient = {
  enable = true;
  server = "https://nebula.example.com";
  enrollCodeFile = "/etc/ncclient/enroll-code";   # or config.sops.secrets.ncclient-enroll.path
};
```

```bash
sudo install -D -m 0600 /dev/stdin /etc/ncclient/enroll-code <<< "XXXXXXXX"
sudo nixos-rebuild switch
journalctl -u ncclient-enroll -u ncclient -b
```

Once the network is enrolled, the enroll unit skips it and the code file isn't read again. A device that was enrolled before v0.8.0 counts as enrolled. `ncclient.service` *requires* `ncclient-enroll.service`, so if an enrollment fails (a used or expired code), the main service doesn't start either until you fix it; see [Re-enroll](#re-enroll).

Without `enrollCodeFile`, enroll with the CLI ([below](#running-ncclient-commands)), or with the desktop app's **Add network** tab.

### Several networks

From v0.8.0, one device can be in several networks, on different servers if needed. Declare each one under `networks`, with its own server and code file:

```nix
services.ncclient = {
  enable = true;
  networks = {
    home   = { server = "https://nebula.example.com"; enrollCodeFile = "/run/secrets/nc-home"; };
    office = { server = "https://nc.office.example";   enrollCodeFile = "/run/secrets/nc-office"; };
  };
};
```

Each is enrolled once, like above. The top-level `server` and `enrollCodeFile` still work and count as a network named `default`, so an existing configuration doesn't need changing; add `networks` entries next to it to join more.

Removing an entry from `networks` doesn't leave that network. Run `sudo ncclient networks remove NAME` for that. See [Several networks on one device](/docs/usage/ncclient/usage/networks/) for what else changes with more than one network.

## Run and control

```bash
systemctl status ncclient
sudo systemctl restart ncclient
journalctl -u ncclient -f
```

Change `server`, `interval`, or `acceptDns` in your configuration and `nixos-rebuild switch`. The unit is restarted with the new values.

## Running ncclient commands

The module puts an `ncclient` wrapper on the system PATH that already points at the service's `stateDir` and `outputDir`. Commands that change the service's state just need root:

```bash
sudo ncclient …
```

## Subnet routes and exit nodes

```bash
sudo ncclient routes list
sudo ncclient routes accept 192.168.1.0/24
```

`reject`, `accept-exit-node --via <IP>`, and `reject-exit-node` work the same way (see the [CLI page](/docs/usage/ncclient/usage/cli/#subnet-routes-and-exit-nodes)). With several networks, name one before the subcommand: `sudo ncclient routes --network NAME list`. The service applies changes on its next poll. With `services.ncclient-desktop` enabled, you can use the network's page in the app instead.

## Split-horizon DNS

Set `services.ncclient.acceptDns = true;` and rebuild. The service then runs with `--accept-dns` and configures the host's resolver (systemd-resolved is the usual case on NixOS). See [Split-horizon DNS](/docs/usage/ncclient/usage/cli/#split-horizon-dns) on the CLI page for details.

## Re-enroll

### With `enrollCodeFile`

The enroll unit only enrolls a network that has **no marker file** yet, so remove that network's marker first:

1. Get a new code from **Nodes → Enroll**, and put it in the file the network's `enrollCodeFile` points at. For a sops-managed secret, update the secret and `nixos-rebuild switch`. For a plain file, overwrite it.
2. Remove the network's marker (`default` for the top-level `enrollCodeFile`, otherwise the name under `networks`):

   ```bash
   sudo rm /var/lib/ncclient/.enrolled-default
   ```

3. Restart **both** units:

   ```bash
   sudo systemctl restart ncclient-enroll ncclient
   ```

4. Check that it worked:

   ```bash
   journalctl -u ncclient-enroll -u ncclient -b
   ```

Restarting only `ncclient` isn't enough. `ncclient-enroll` is a oneshot with `RemainAfterExit=true`, so it stays "active" from boot and isn't run again.

A new code for the same node replaces that network's token; the device stays connected with the old one until then. If you're moving to a **different server**, change `server` and rebuild in step 1. Enrolling with the new server adds a new network next to the old one, so remove the old one afterwards with `sudo ncclient networks remove NAME`.

### Without `enrollCodeFile`

Enroll with the CLI, pointed at the service's state (see [Running ncclient commands](#running-ncclient-commands)). A new code for the same node on the same server overwrites that network's token, so there's nothing to delete:

```bash
sudo ncclient --server https://nebula.example.com enroll --code NEWCODE
```

The service switches to the new token on its next poll. If you changed servers, the enroll adds a new network; remove the old one with `sudo ncclient networks remove NAME`.

### With the desktop app

If `services.ncclient-desktop` is enabled, the app's **Add network** tab works as described on the [Linux App page](/docs/usage/ncclient/usage/linux/#re-enroll), with no password prompt for members of `adminGroups`.

## Updates

The client can check daily for new releases and tell you when one is out (`sudo ncclient auto-update enable`). On NixOS it never installs anything itself; you update the flake input and rebuild. See [Automatic updates](/docs/usage/ncclient/usage/auto-update/).

## Troubleshooting

- **`ncclient.service` won't start and `ncclient-enroll` failed**: a network isn't enrolled yet, and its code file is missing or its code was rejected (already used or expired). `journalctl -u ncclient-enroll -b` says which. Put a fresh code in the file and restart both units.
- **`ncclient: command not found`**: expected; see [Running ncclient commands](#running-ncclient-commands).
- **`Revoked, deleted, or re-enrolled elsewhere - Nebula stopped. Enroll again to reconnect.`**: the device was revoked, deleted, or re-enrolled elsewhere; Nebula was stopped and its config and key removed. [Re-enroll](#re-enroll) it with a new code.
- **Enrolled with the CLI but the service still waits for enrollment**: the CLI ran without both environment variables, so the token went to root's home directory. Run it again with them.
