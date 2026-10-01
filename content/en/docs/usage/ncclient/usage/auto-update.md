---
title: Automatic updates
linkTitle: Automatic updates
weight: 60
---

From v0.7.0, the client can keep itself up to date. It's **off by default**, and only an administrator on the device itself can turn it on. The Nebula Commander server shows each node's client version and whether automatic updates are on, but can't change the setting.

| Install | What "on" does |
|---|---|
| Windows (MSI) | The service downloads and installs new releases during a daily window you choose. |
| Linux deb/rpm from the [package repository](/docs/usage/ncclient/installation/linux/) | A systemd timer upgrades the Nebula Commander packages from that repository during the window. |
| NixOS module | **Notify only.** Checks daily and tells you when a release is out, with the commands to update. It never changes the system: your flake decides the version. |
| Docker, Flatpak, pip, a bare binary | Not available. Update these the way you installed them (pull a new image, Flathub, `pip install -U`). |

## Turn it on

**Windows:** open the app as administrator (**Relaunch as administrator**), go to **Settings → Updates**, switch on **Automatic updates**, pick the window and click **Save**. Or, from an elevated prompt:

```powershell
ncclient auto-update enable --window 02:00-05:00
```

**Linux:** in the desktop app, **Settings → Updates**, then **Apply** (you need to be in the `sudo` or `wheel` group). Or:

```bash
sudo ncclient auto-update enable --window 02:00-05:00
```

This is refused if the official package repository isn't configured. [Add it](/docs/usage/ncclient/installation/linux/) first. A `.deb` or `.rpm` installed by hand without the repository can't update itself.

**NixOS:** `sudo ncclient auto-update enable`, or the desktop app. The window doesn't apply.

Other commands:

```bash
ncclient auto-update status      # installed version, setting, last check and install
ncclient auto-update check-now   # check now, and install if automatic updates are on (admin)
ncclient auto-update disable     # admin
```

## The window

Updates install once a day, at a random time inside the window (local time, `HH:MM-HH:MM`, default `02:00-05:00`). Windows that cross midnight, like `23:00-01:00`, work. A device that's off during the window waits for the next one. The tunnel drops for a few seconds while the client restarts.

## What it trusts

- **Windows and NixOS** read a release manifest from `https://pkgs.nebulacommander.com/updates/latest.json`. The client uses it only if its signature checks out against a key built into the client. The Windows installer it names must come from the project's GitHub releases and match the manifest's SHA-256, or it's discarded.
- **Linux packages** come from the signed package repository, with the usual apt/dnf/zypper signature checks. If the repository can't be verified, the update stops and says so.
- The client only ever moves to a **newer stable** release. It never downgrades or installs pre-releases. Development builds never update.

## Where to look when something goes wrong

- `ncclient auto-update status` shows the last check and the last install, with the error if there was one.
- **Windows:** the installer log is in `C:\ProgramData\nebula-commander\updates\install-<version>.log` (administrators only). If the app was open during an update, it shows **Nebula Commander was updated → Restart now**.
- **Linux:** `journalctl -u ncclient-update`. The timer is `ncclient-update.timer`; its window lives in `/etc/systemd/system/ncclient-update.timer.d/window.conf`, which ncclient writes. Don't edit it by hand.

## NixOS: updating when notified

When a release is out, `ncclient auto-update status`, the desktop app and a desktop notification tell you. To update:

```bash
nix flake update nebula-commander     # in your system flake
sudo nixos-rebuild switch
```

If your flake pins a release (`github:NixRTR/nebula-commander/v0.7.0`), change the tag instead. A NixOS build reports the commit it was built from (for example "built from commit f67bc81") rather than a version number.
