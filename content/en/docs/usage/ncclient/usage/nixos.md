---
title: NixOS
linkTitle: NixOS
weight: 30
---

This page covers day-to-day use of the `services.ncclient` module. See [NixOS installation](/docs/usage/ncclient/installation/nixos/) for importing the module and its full option list. Paths below use the defaults (`stateDir = "/var/lib/ncclient"`, `outputDir = "/var/lib/ncclient/nebula"`); adjust them if you changed those options.

## What the module sets up

- `ncclient.service`: runs `ncclient run` as root, with `nebula` from `nebulaPackage`, plus `nftables` and `iproute2` (for subnet-router/exit-node forwarding and NAT), on its PATH. It restarts on failure.
- `ncclient-enroll.service`: only when `enrollCodeFile` is set. A oneshot that runs `ncclient enroll` before the main service, **only if no token exists yet**.
- The D-Bus policy and polkit rules the [Linux desktop app](/docs/usage/ncclient/usage/linux/) uses, if you also enable `services.ncclient-desktop`. Only members of `adminGroups` (default `wheel`/`sudo`) can make changes without a password.
- An `ncclient` command on the system PATH, pre-pointed at the service's state. See [Running ncclient commands](#running-ncclient-commands).

## Where state lives

| Path | What it is |
|------|------------|
| `/var/lib/ncclient/token` | Device token |
| `/var/lib/ncclient/settings.json` | Server URL, node ID, accepted routes |
| `/var/lib/ncclient/nebula/` | Nebula `config.yaml`, certificates, status |

Both directories are `0700 root:root`.

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

Once a token exists, the enroll unit does nothing and the code file isn't read. If the token is ever lost, though, the unit tries the used-up code and fails. `ncclient.service` *requires* `ncclient-enroll.service`, so the main service won't start either until you [re-enroll](#re-enroll).

Without `enrollCodeFile`, enroll with the CLI ([below](#running-ncclient-commands)), or with the desktop app's Enrollment tab.

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

`reject`, `accept-exit-node --via <IP>`, and `reject-exit-node` work the same way (see the [CLI page](/docs/usage/ncclient/usage/cli/#subnet-routes-and-exit-nodes)). The service applies changes on its next poll. With `services.ncclient-desktop` enabled, you can use the app's Status tab instead.

## Split-horizon DNS

Set `services.ncclient.acceptDns = true;` and rebuild. The service then runs with `--accept-dns` and configures the host's resolver (systemd-resolved is the usual case on NixOS). See [Split-horizon DNS](/docs/usage/ncclient/usage/cli/#split-horizon-dns) on the CLI page for details.

## Re-enroll

### With `enrollCodeFile`

The enroll unit only enrolls when there is **no token**, so remove the old one first:

1. Get a new code from **Nodes → Enroll**, and put it in the file `enrollCodeFile` points at. For a sops-managed secret, update the secret and `nixos-rebuild switch`. For a plain file, overwrite it.
2. Remove the old token:

   ```bash
   sudo rm /var/lib/ncclient/token
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

Keep the old token until you have the new code; once it's removed the device is offline until enrollment succeeds. If you're moving to a different server, change `server` and rebuild in step 1.

### Without `enrollCodeFile`

Enroll with the CLI, pointed at the service's state (see [Running ncclient commands](#running-ncclient-commands)). It overwrites the old token, so there's nothing to delete:

```bash
sudo ncclient --server https://nebula.example.com enroll --code NEWCODE
```

The service switches to the new token on its next poll. If you changed servers, update `server` and rebuild, which also restarts the service.

### With the desktop app

If `services.ncclient-desktop` is enabled, the app's **Enrollment** tab works as described on the [Linux App page](/docs/usage/ncclient/usage/linux/#re-enroll), with no password prompt for members of `adminGroups`.

## Troubleshooting

- **`ncclient.service` won't start and `ncclient-enroll` failed**: there's no token, and the code file is missing or its code was rejected (already used or expired). Put a fresh code in the file and restart both units.
- **`ncclient: command not found`**: expected; see [Running ncclient commands](#running-ncclient-commands).
- **`Token invalid or expired. Waiting for re-enrollment...`**: the node was re-enrolled elsewhere or its token was revoked. [Re-enroll](#re-enroll).
- **Enrolled with the CLI but the service still waits for enrollment**: the CLI ran without both environment variables, so the token went to root's home directory. Run it again with them.
