# home-edge-router

This repository contains the configuration for my home edge router / ingress stack. It provides HTTPS/SSL termination with valid Let's Encrypt certificates via Cloudflare DNS challenge, and reverse-proxies traffic to internal home services. It also exposes raw TCP/UDP ports for media streaming protocols.

## Services

| Service | Image | Role |
| ------- | ----- | ---- |
| `caddy` | `ghcr.io/gitgc/home-proxy-webserver:main` | HTTPS reverse proxy with Cloudflare DNS ACME |
| `haproxy` | `haproxy:lts-alpine` | TCP relay for RTSP (8554) and WebRTC (8555) |
| `webrtc-udp` | `alpine/socat` | UDP relay for WebRTC (8555 UDP → Frigate) |
| `netdata` | `netdata/netdata` | Host-level system monitoring, reports to Netdata Cloud |
| `autoheal` | `willfarrell/autoheal` | Restarts unhealthy containers automatically |

## Requirements

* Docker

## Setup

Copy `.env.example` to `.env` and replace all `CHANGE_ME` values:

| Environment Variable           | Description                                                        |
| ------------------------------ | ------------------------------------------------------------------ |
| `CADDY_DOMAIN`                 | Primary domain (must be managed in Cloudflare DNS)                 |
| `CADDY_HEALTH_CHECK_URI`       | URI Caddy uses for its own health check                            |
| `CF_AUTH_TOKEN`                | Cloudflare API token for ACME DNS challenge                        |
| `CF_ZONE_ID`                   | Cloudflare Zone ID for the domain                                  |
| `FRIGATE_UPSTREAM_HOST`        | Hostname for the Frigate NVR server                                |
| `HOME_ASSISTANT_UPSTREAM_HOST` | Hostname for the Home Assistant server                             |
| `UNIFI_UPSTREAM_HOST`          | Hostname or IP for the Unifi OS Console                            |
| `PLEX_UPSTREAM_HOST`           | Hostname or IP for the Plex Media Server                           |

Then start the stack:

```console
docker compose up -d
```

## Endpoints

### HTTPS (via Caddy)

| Endpoint                              | Upstream                          | Description              |
| ------------------------------------- | --------------------------------- | ------------------------ |
| `https://home.$CADDY_DOMAIN`          | `$HOME_ASSISTANT_UPSTREAM_HOST`   | Home Assistant dashboard |
| `https://cam.$CADDY_DOMAIN`           | `$FRIGATE_UPSTREAM_HOST`          | Frigate NVR              |
| `https://network.$CADDY_DOMAIN`       | `$UNIFI_UPSTREAM_HOST`            | Unifi OS Console         |
| `https://plex.$CADDY_DOMAIN`          | `$PLEX_UPSTREAM_HOST:32400`       | Plex Media Server        |
| `https://uptime.$CADDY_DOMAIN`        | `$HOME_ASSISTANT_UPSTREAM_HOST:3001` | Uptime Kuma           |
| `https://music.$CADDY_DOMAIN`         | `$HOME_ASSISTANT_UPSTREAM_HOST:8095` | HifiBerry / music     |
| `https://netdata.edge.$CADDY_DOMAIN`  | `host.docker.internal:19999`      | Netdata monitoring       |

### TCP/UDP (via HAProxy + socat)

| Port / Protocol | Destination              | Description       |
| --------------- | ------------------------ | ----------------- |
| `8554 TCP`      | `$FRIGATE_UPSTREAM_HOST:8554` | RTSP streaming |
| `8555 TCP`      | `$FRIGATE_UPSTREAM_HOST:8555` | WebRTC TCP     |
| `8555 UDP`      | `$FRIGATE_UPSTREAM_HOST:8555` | WebRTC UDP     |
