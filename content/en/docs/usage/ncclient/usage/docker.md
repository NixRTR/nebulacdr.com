---
title: Docker
linkTitle: Docker
weight: 10
---

The Docker client runs ncclient, Nebula, and (on lighthouses) dnsmasq in one container. See [Docker installation](/docs/usage/ncclient/installation/docker/) for the image and a full `docker-compose.yml`. The commands below assume that compose file, with the service named `ncclient`, run from the directory that holds it.

## Where state lives

Everything that must survive a container recreate lives in the `/data` volume, as long as the compose file sets these two paths (the example does):

```yaml
NEBULA_OUTPUT_DIR: "/data/nebula"
NEBULA_DEVICE_TOKEN_FILE: "/data/nebula-commander/token"
```

| Path in the container | What it is |
|------|------------|
| `/data/nebula-commander/token` | Device token |
| `/data/nebula-commander/settings.json` | Server URL, node ID, accepted routes |
| `/data/nebula/` | Nebula `config.yaml`, certificates, status |

Without those two variables the image falls back to `/etc/nebula` and `/etc/nebula-commander/token`. Those paths are **outside** the volume, so the device loses its enrollment every time the container is recreated.

## Enroll

1. In Nebula Commander, create or sign a certificate for the node, then click **Enroll** and copy the code.
2. Set `ENROLL_CODE` in the compose file and start the container:

   ```bash
   docker compose up -d
   docker compose logs -f ncclient
   ```

The container enrolls once, when no token exists yet. After that, `ENROLL_CODE` is ignored and you can remove it.

## Run and control

The container starts ncclient on its own and restarts with Docker (`restart: unless-stopped`).

```bash
docker compose logs -f ncclient     # ncclient, Nebula, and dnsmasq output
docker compose restart ncclient     # restart everything in the container
docker compose down                 # stop (the volume and enrollment are kept)
```

`docker compose up -d` again after editing the compose file.

## Subnet routes and exit nodes

Run `ncclient routes` inside the container. Pass `NEBULA_COMMANDER_CONFIG_DIR` so the command sees the same `settings.json` as the running client, and `--output-dir` so it sees the routes offered to the node:

```bash
docker compose exec -e NEBULA_COMMANDER_CONFIG_DIR=/data/nebula-commander ncclient \
  ncclient routes --output-dir /data/nebula list

docker compose exec -e NEBULA_COMMANDER_CONFIG_DIR=/data/nebula-commander ncclient \
  ncclient routes --output-dir /data/nebula accept 192.168.1.0/24
```

`reject`, `accept-exit-node --via <IP>`, and `reject-exit-node` work the same way (see the [ncclient CLI page](/docs/usage/ncclient/usage/cli/#subnet-routes-and-exit-nodes)). The running client applies changes on its next poll.

## Split-horizon DNS

The Docker client is how a **lighthouse serves** [Magic DNS](/docs/web-ui/dns/). Set `SERVE_DNS: "true"`. When the node is a lighthouse and DNS is enabled for the network, the container polls for the dnsmasq config (every `NEBULA_DNS_POLL_INTERVAL` seconds, default 60) and runs dnsmasq on the node's Nebula IP, port 53. On non-lighthouses dnsmasq stays off.

The container does **not** apply split-horizon DNS to the host it runs on (it doesn't pass `--accept-dns`). To resolve Nebula names on that host too, point its resolver at the lighthouse IP yourself.

## Re-enroll

The container only uses `ENROLL_CODE` when no token exists. There are two ways to re-enroll.

**Enroll inside the running container.** This needs no restart unless the server changed:

```bash
docker compose exec -e NEBULA_COMMANDER_CONFIG_DIR=/data/nebula-commander ncclient \
  ncclient enroll --code NEWCODE
```

The server URL comes from `NEBULA_COMMANDER_SERVER`. The old token is replaced, and the client switches to the new one on its next poll.

**Or remove the token and let the container enroll on start:**

```bash
docker compose exec ncclient rm /data/nebula-commander/token
# set ENROLL_CODE to the new code in docker-compose.yml, then:
docker compose up -d --force-recreate
docker compose logs -f ncclient
```

To move to a **different server**, change `NEBULA_COMMANDER_SERVER`, use either method above, then `docker compose up -d --force-recreate`.

## Troubleshooting

- **`Token file not found and ENROLL_CODE not set`**: the container has no token and nothing to enroll with. Set `ENROLL_CODE`, or check that the volume is mounted and the token path points into it.
- **The device keeps asking to enroll after every recreate**: the token path isn't in the volume. See [Where state lives](#where-state-lives).
- **`Token invalid or expired. Waiting for re-enrollment...`**: the node was re-enrolled elsewhere, or its token was revoked. [Re-enroll](#re-enroll).
- **Nebula can't create its tun device**: the container needs `network_mode: host` and `cap_add: NET_ADMIN`. It also needs **rootful** Docker. Under rootless Docker or Podman, "host" networking is a private namespace owned by an unprivileged user, so Nebula can't configure the real host's network and inbound UDP never reaches it.
- **dnsmasq never starts**: check `SERVE_DNS` is set, the node is a lighthouse, and DNS is enabled for the network. The logs show `Warning: listen-address … not assigned` if Nebula's interface never came up.
