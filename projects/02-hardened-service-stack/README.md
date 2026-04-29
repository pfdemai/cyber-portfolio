# 02 — Hardened Service Stack

A Docker Compose stack running on an isolated VM (vmbr1).
This document explains the architectural decisions behind the deployment —
service isolation, network namespacing, and secrets handling.

---

## Stack Overview

| Service | Purpose | Port |
|---------|---------|------|
| Jellyfin | Media streaming server | 8096 |
| Radarr | Movie library management | 7878 |
| Sonarr | TV series management | 8989 |
| Prowlarr | Indexer manager (feeds Radarr/Sonarr) | 9696 |
| qBittorrent | Download client | 8080 |

All services share a single internal Docker bridge network (`media_net`, 172.20.0.0/24).
No service is reachable from outside the VM except through Tailscale.

<!-- INPUT: Add NordVPN and any additional services to the table above once confirmed. -->

---

## Service Isolation

Docker's internal networking means services communicate by container name, not by
host IP. Jellyfin never needs to know where Radarr is on the physical network —
it addresses it as `radarr:7878` within `media_net`.

Port bindings (`ports:`) expose services on the host VM's IP. Since the VM sits on
vmbr1 (isolated lab network), these ports are only reachable from the Tailscale mesh
or a machine on vmbr1 — never directly from WAN.

---

## Secrets Handling

Configuration uses a `.env` file with restricted filesystem permissions (600).
The file is excluded from version control via `.gitignore`.

See [configs/.env.example](configs/.env.example) for the variable structure.

**Why not Infisical yet:** Infisical is the next step — centralised secrets management
with audit logs and rotation. The `.env` approach is acceptable for a single-VM homelab
but does not scale to multiple services or rotate credentials automatically.
The current setup is documented honestly: it's a starting point, not a finished security posture.

---

## Volumes and Persistence

Jellyfin config is stored in a Docker named volume (`jellyfin_config`), not a bind mount to the SSD.
See [decisions/ssd-passthrough.md](decisions/ssd-passthrough.md) for why.

---

## Configuration

See [configs/docker-compose.yml](configs/docker-compose.yml) for the full stack definition.
