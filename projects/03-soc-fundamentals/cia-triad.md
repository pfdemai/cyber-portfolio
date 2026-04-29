# CIA Triad — Applied to the Homelab

The CIA triad (Confidentiality, Integrity, Availability) is the foundational model
for evaluating security posture. Here's where each property lives in this infrastructure.

---

## Confidentiality

*Preventing unauthorized access to data.*

**In this infrastructure:**
- Tailscale encrypts all remote access traffic end-to-end (WireGuard). No credentials travel in plaintext.
- `.env` file has permissions set to 600 — readable only by the service user.
- vmbr1 isolation means media VM services are not exposed on the LAN or WAN interfaces.
- pfSense's LAB→LAN block rule ensures the lab zone cannot reach management interfaces.

**Weak point:** No encryption at rest on the SSD. Media files are accessible to anyone with physical access to the drive.

---

## Integrity

*Ensuring data is not altered without authorization.*

**In this infrastructure:**
- Proxmox snapshots create point-in-time recovery points before configuration changes.
- Docker named volumes are managed by the QEMU layer — writes are consistent even on VM restart.
- Jellyfin config on the VM disk (not SSD) protects the config database from corruption on SSD disconnect.

**Weak point:** No file integrity monitoring (FIM) is currently in place. A compromised container could modify config files without detection. This is a gap Wazuh will address when deployed.

---

## Availability

*Ensuring services are accessible when needed.*

**In this infrastructure:**
- `restart: unless-stopped` on all Docker services — automatic recovery from container crashes.
- Tailscale mesh means remote access survives ISP IP changes (no dependency on a static WAN IP).
- Proxmox allows live VM management without rebooting the host.

**Weak point:** Single host — no redundancy. A hardware failure on the MacBook takes down all services. Acceptable for a homelab; unacceptable in a production SOC environment.
