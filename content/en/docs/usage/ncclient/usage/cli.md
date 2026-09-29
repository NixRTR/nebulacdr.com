---
title: ncclient CLI
linkTitle: ncclient CLI
weight: 20
---

This page covers the plain `ncclient` command from the [binaries](/docs/usage/ncclient/installation/binaries/) or [Pip](/docs/usage/ncclient/installation/pip/), run by hand or under your own init system. If you installed the Linux packages, the Windows MSI, or the NixOS module, use the [Linux App](/docs/usage/ncclient/usage/linux/), [Windows App](/docs/usage/ncclient/usage/windows/), or [NixOS](/docs/usage/ncclient/usage/nixos/) page instead. Those set up a background service that keeps its state somewhere the plain CLI doesn't look.

## Where state lives

| What | Default location |
|------|------------------|
| Device token | The OS keyring if the Python `keyring` package is available. Otherwise `~/.nebula/device-token` (the frozen binaries and a plain `pip install` use this). Set `NEBULA_DEVICE_TOKEN_FILE` to use a specific file. |
| `settings.json` (server, node ID, accepted routes) | `~/.config/nebula-commander/` on Linux and macOS, `%APPDATA%\nebula-commander\` on Windows. Set `NEBULA_COMMANDER_CONFIG_DIR` to change it. |
| Nebula config and certificates | `/etc/nebula` on Linux and macOS, `%USERPROFILE%\.nebula` on Windows. Set with `--output-dir`. |

`~` is the home directory of whoever runs the command, so `sudo ncclient …` uses root's (`/root/.nebula/device-token`). **Run `enroll`, `run`, and `routes` as the same user** (and with the same environment variables), or they won't find each other's files.

## Enroll

1. In Nebula Commander, open **Nodes**, select the node for this device, and click **Enroll**.
2. Copy the enrollment code.
3. On the device, as the user that will run the daemon (root on Linux):

```bash
sudo ncclient enroll --server https://YOUR_NEBULA_COMMANDER_URL --code XXXXXXXX
```

## Run (daemon)

After enrollment, run ncclient so it periodically pulls config and certificates and runs or restarts Nebula:

```bash
sudo ncclient run --server https://YOUR_NEBULA_COMMANDER_URL
```

Defaults: poll every 60 seconds, write files to `/etc/nebula` (or `~/.nebula` on Windows), and start or restart Nebula from PATH when config changes. `--server` can be left out once you've enrolled; ncclient uses the server saved in `settings.json`, or `NEBULA_COMMANDER_SERVER`.

**Linux:** creating the Nebula TUN device requires root, so run `run` with `sudo`.

### Options

| Option | Description |
|--------|-------------|
| `--output-dir DIR` | Where to write `config.yaml`, `ca.crt`, `host.crt` (default: `/etc/nebula` on Linux/macOS, `~/.nebula` on Windows) |
| `--interval N` | Poll interval in seconds (default: 60) |
| `--nebula PATH` | Path to the `nebula` binary if it is not on PATH |
| `--restart-service NAME` | Instead of running nebula directly, restart this systemd service (e.g. `nebula`). Use only one of `--nebula` or `--restart-service`. |
| `--accept-dns` | Apply split-horizon DNS. See [Split-horizon DNS](#split-horizon-dns). |

Example with nebula in a non-standard location:

```bash
sudo ncclient run --server https://nc.example.com --nebula /usr/local/bin/nebula
```

Example using systemd to run Nebula (ncclient only restarts the service):

```bash
sudo ncclient run --server https://nc.example.com --restart-service nebula
```

**Certificates:** if the cert was **created** via the server (Create certificate in the UI), the bundle includes `host.key`. If it was **signed** (Sign flow), the server doesn't have the key, so put your `host.key` in the output directory.

## Run as a service

### Linux (`ncclient install`)

```bash
sudo ncclient install
```

This checks that root is enrolled (if not, it prints the `ncclient enroll …` command to run first). It then prompts for the server URL and options, writes `/etc/default/ncclient` and `/etc/systemd/system/ncclient.service`, and enables (and optionally starts) the service. The unit runs `ncclient run` as root.

- Use `--no-start` to enable without starting.
- Use `--non-interactive` with `NEBULA_COMMANDER_SERVER` (and optional env vars) set for scripting.

{{% alert title="Current limitation" color="warning" %}}
Of the values `ncclient install` writes to `/etc/default/ncclient`, only `NEBULA_COMMANDER_SERVER` currently takes effect. The service runs with the default output directory (`/etc/nebula`), interval, and `nebula` from PATH. To change those, add the flags to `ExecStart` with `sudo systemctl edit --full ncclient`.
{{% /alert %}}

Day-to-day:

```bash
sudo systemctl status ncclient
sudo systemctl restart ncclient
journalctl -u ncclient -f
```

### Other platforms

Run `ncclient run` under your init system (launchd on macOS, Task Scheduler or NSSM on Windows). Example configs are in the repo under `client/examples/`. See [README-startup.md](https://github.com/NixRTR/nebula-commander/blob/main/client/examples/README-startup.md) for step-by-step setup on macOS and Windows. On Windows, the [MSI's service](/docs/usage/ncclient/usage/windows/) is usually the easier option.

## Subnet routes and exit nodes

A [subnet router or exit node](/docs/usage/unsafe-routes/) that an admin assigns to this node is only *offered* to it. It isn't used until you accept it on the device. Use `ncclient routes`, as the same user as `run` and with the same `--output-dir`:

```bash
sudo ncclient routes list                              # offered routes, and which are accepted
sudo ncclient routes accept 192.168.1.0/24             # add --via <IP> if several gateways offer it
sudo ncclient routes reject 192.168.1.0/24
sudo ncclient routes accept-exit-node --via 10.100.0.1
sudo ncclient routes reject-exit-node
```

You can accept several subnet routes as long as they don't overlap, and at most one exit node. The running daemon picks up changes on its next poll; no restart needed.

## Split-horizon DNS

When the server has [DNS enabled](/docs/web-ui/dns/) for the network, pass `--accept-dns` to `run`. ncclient then fetches the DNS config (domain and lighthouse IPs) and configures the host to resolve the Nebula domain via the network's DNS. On Linux the client detects how the host manages DNS and configures it to match:

- **systemd-resolved** (on its own, or behind NetworkManager): per-link DNS on the Nebula interface, so only the Nebula domain goes to its servers.
- **NetworkManager without systemd-resolved** (e.g. Debian desktops): NetworkManager's dnsmasq plugin with a rule for the Nebula domain. If NetworkManager is writing `/etc/resolv.conf` itself, the client switches it to the plugin; that needs `dnsmasq` (Debian: `dnsmasq-base`) installed.
- **A standalone dnsmasq service**: a rule in `/etc/dnsmasq.d/`.
- **Plain `/etc/resolv.conf`**: last resort, not true split-horizon.

It also keeps NetworkManager from taking over the Nebula interface, re-checks the DNS on every poll (re-applying it if something undid it), and removes everything when it stops or DNS is turned off. If none of these can work, the client logs why and what to install. On Windows it uses NRPT. Applying DNS needs root (Linux) or Administrator (Windows). To remove the DNS override, stop ncclient normally (e.g. Ctrl+C or `systemctl stop`).

## Re-enroll

Get a new code from **Nodes → Enroll**, then run `enroll` again **as the same user, with the same environment variables** as the daemon:

```bash
sudo ncclient enroll --server https://YOUR_NEBULA_COMMANDER_URL --code NEWCODE
```

This overwrites the old token and node ID. The running daemon switches to the new token on its next poll. If you changed servers, restart it too (`sudo systemctl restart ncclient` for an `ncclient install` service) and update `NEBULA_COMMANDER_SERVER` in `/etc/default/ncclient`.

If `enroll` succeeds but the daemon keeps logging `Token invalid or expired`, the two are using different token locations (usually one ran with `sudo` and the other didn't). See [Where state lives](#where-state-lives).

## Troubleshooting

- **No TUN device / cannot ping Nebula IP**: on Linux, run with `sudo`. For the Sign flow, make sure `host.key` is in the output directory. Check Nebula's error output (e.g. "failed to get tun device", "no such file").
- **Nebula starts then exits**: often a missing `host.key` (Sign flow), a wrong config path, or on Linux not running as root. Check the Nebula lines ncclient prints.
- **`Token not found. Run 'ncclient enroll' first.`**: this user has no token. Enroll as the same user that runs the daemon.

## macOS notes

Default output dir: `/etc/nebula`; for non-root use `--output-dir ~/.nebula`. Nebula: use PATH or `--nebula /opt/homebrew/bin/nebula` (Apple Silicon) or `/usr/local/bin/nebula` (Intel). Don't use `--restart-service`; use launchd for background runs.

## Windows notes

Default output dir: `%USERPROFILE%\.nebula`. Use `--nebula` if `nebula.exe` is not on PATH. Don't use `--restart-service`. The plain CLI's token and settings are separate from the shared `%ProgramData%\nebula-commander\` location used by the MSI's app and service. If you installed the MSI, use the [Windows App](/docs/usage/ncclient/usage/windows/) to enroll instead.
