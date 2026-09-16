---
title: Docker
linkTitle: Docker
weight: 10
---

The Docker client is the **preferred** method for the **first lighthouse** in a network so the container can run dnsmasq and you can use [Magic DNS](/docs/web-ui/dns/) (split-horizon DNS) for the network. Other devices (CLI, tray, or additional Docker clients) can then resolve Nebula hostnames via the lighthouse.

**Image:** `ghcr.io/nixrtr/nebula-commander-ncclient:latest`, or build from the repo `client/docker` (Dockerfile in that directory).

**Required environment:**

- `NEBULA_COMMANDER_SERVER` – Base URL of your Nebula Commander backend (e.g. `https://nc.example.com`), no trailing slash.

**Optional environment:**

- `ENROLL_CODE` – One-time enrollment code from the Nebula Commander UI (Nodes → Enroll for the node). Only used when the token file does not exist; after enrollment the token is stored and this is ignored.
- `SERVE_DNS` – Set to `"true"` to run dnsmasq on this node when it is a lighthouse, so the network can use Magic DNS. Omit or set to `false` if this node is not a lighthouse or you do not need DNS.
- `NEBULA_DNS_POLL_INTERVAL` – Seconds between dnsmasq config polls when this node is a lighthouse (default: 60).
- `NEBULA_OUTPUT_DIR` – Directory where ncclient writes Nebula config and certs inside the container (default: `/data/nebula`).
- `NEBULA_DEVICE_TOKEN_FILE` – Path to the device token file (default: `/data/nebula-commander/token`).

Use a **persistent volume** for `/data` so the token and Nebula config/certs survive restarts. The compose file uses `network_mode: host` so Nebula and dnsmasq can bind to the host.

**Example (docker-compose):**

```yaml
services:
  ncclient:
    image: ghcr.io/nixrtr/nebula-commander-ncclient:latest
    network_mode: host
    restart: unless-stopped
    environment:
      NEBULA_COMMANDER_SERVER: "https://nc.example.com"
      ENROLL_CODE: "XXXXXXXX"   # one-time, from UI
      SERVE_DNS: "true"         # for first lighthouse + Magic DNS
    volumes:
      - ncclient-data:/data

volumes:
  ncclient-data:
    driver: local
```

**Steps:**

1. In Nebula Commander, create a network and add a node for this device. Mark it as a **lighthouse** if this will be the first lighthouse and you want Magic DNS.
2. Create or sign a certificate for the node, then click **Enroll** and copy the one-time code.
3. Set `NEBULA_COMMANDER_SERVER` and `ENROLL_CODE` (and `SERVE_DNS: "true"` for the first lighthouse), then start the container.
4. After enrollment, the container fetches config and certs and runs Nebula (and dnsmasq if `SERVE_DNS` is set and the node is a lighthouse).
