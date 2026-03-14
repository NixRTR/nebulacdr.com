---
title: Running Nebula manually
linkTitle: nebula
weight: 30
---

This is the **manual** method: you use Nebula Commander to create networks, nodes, and certificates, then run [Nebula](https://github.com/slackhq/nebula) on devices yourself without ncclient. Config and certs are copied or downloaded from the UI (or API) and you start Nebula manually. Use this when you prefer to deploy config and certs yourself or when ncclient is not available (e.g. on mobile).

## When to use this

- You prefer to deploy config and certs yourself (copy to the device, run `nebula -config ...`).
- You do not want to enroll devices or run the ncclient daemon.
- You are fine updating config and certs manually when the network or node changes (re-download from the UI or API and replace files, then restart Nebula).

With [ncclient](/docs/usage/ncclient/), the device enrolls once and ncclient polls for config and certs and can run or restart Nebula automatically. With manual setup, you handle file deployment and restarts yourself.

## Steps

### 1. Create network and node in Nebula Commander

In the [Web UI](/docs/web-ui/): create a network, add a node for this device, and create or sign a certificate for the node.

- **Create certificate** – The server generates the key and cert; you can download a bundle that includes `host.key`, `host.crt`, `ca.crt`, and config.
- **Sign certificate** – You generate the key on the device; the server signs the cert. You will need to place your own `host.key` next to the downloaded certs.

### 2. Get config and certs onto the device

Download or copy from the UI (or use the API) the node's config and certificate files. You typically need:

- `config.yaml` (Nebula config for this node)
- `ca.crt` (CA certificate)
- `host.crt` (host certificate for this node)
- `host.key` (only if you used Create certificate; with Sign, you already have this on the device)

Where to get them depends on your Nebula Commander version: use the node's detail or download actions in the UI, or the device/config API. Place the files in a directory on the device (e.g. `/etc/nebula` or `~/.nebula`).

### 3. Install and run Nebula on the device

Install Nebula from [slackhq/nebula](https://github.com/slackhq/nebula) (packages, binary release, or build from source). Then run:

```bash
nebula -config /path/to/config.yaml
```

Use the path to the `config.yaml` you deployed. Nebula will read `ca.crt`, `host.crt`, and `host.key` from the paths specified in the config (often the same directory as the config).

### 4. Run Nebula at startup (optional)

Use your platform's init system so Nebula keeps running: systemd on Linux, launchd on macOS, or a Windows service/task. When you change config or certs (after re-downloading from Nebula Commander), replace the files and restart Nebula.

## Mobile devices

**The official Nebula client from [defined.net](https://defined.net)** (Nebula app) is the only way to run Nebula on a mobile device for now. ncclient is not available on mobile. Use the defined.net app and deploy config and certs manually: create the node and certificate in Nebula Commander, download or copy the config and cert files (e.g. from the Web UI or API), then import or place them in the app as the defined.net client expects.

## Summary

| | ncclient | Manual (nebula) |
|--|----------|-----------------|
| Enrollment | One-time; device gets a token | None |
| Config/certs | Fetched automatically by ncclient | You copy or download and place them |
| Nebula process | ncclient can run or restart it | You run and restart Nebula yourself |
| Updates | ncclient polls and updates files | You re-download and replace files, then restart Nebula |
