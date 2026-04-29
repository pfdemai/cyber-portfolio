# Defense in Depth — Applied to the Homelab

Defense in depth means layering controls so that a failure at one layer
does not immediately result in full compromise. Each layer buys time and limits blast radius.

---

## The Layers in This Infrastructure

### Layer 1 — Network Perimeter (pfSense)

pfSense is the first enforcement point. Default deny on WAN. The lab zone (vmbr1) is
blocked from reaching the LAN zone (vmbr0) at the firewall level.

An attacker who compromises a service on vmbr1 hits this wall immediately. They cannot
pivot to the Proxmox management interface or LAN hosts without bypassing pfSense.

### Layer 2 — Remote Access (Tailscale)

Tailscale replaces open WAN ports. No port forwarding, no exposed SSH port, no attack
surface on the WAN interface. Access requires a valid Tailscale identity — authentication
is handled by Tailscale's control plane before a packet reaches the homelab.

### Layer 3 — Service Isolation (Docker networking)

Services communicate over `media_net` (172.20.0.0/24) — an internal bridge network.
No service has direct access to the host network stack. A compromise of the qBittorrent
container gives access to `media_net`, not to the VM's host interface or vmbr1 directly.

### Layer 4 — Secrets Handling (.env, 600 permissions)

Credentials are stored in `.env`, not in `docker-compose.yml` or environment variables
visible to `docker inspect`. File permissions are 600 — the running service user reads
them, nothing else does.

---

## Where the Layers Fail

This is a homelab — the layers are real but thin:
- No IDS/IPS monitoring traffic between layers
- No EDR on the media VM
- No centralized log correlation (Wazuh not yet deployed)

Each gap represents a detection blind spot. A lateral movement attempt between layers
would not currently generate an alert. This is the honest state of the infrastructure.
