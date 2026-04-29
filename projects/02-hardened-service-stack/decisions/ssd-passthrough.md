# Decision: Jellyfin Config on VM Disk, Not SSD

**Status:** Active
**Date:** <!-- INPUT: when this decision was made -->

---

## Context

The Proxmox host has an external SSD passed through to the media VM for media storage
(films, series). The SSD offers significantly more capacity than the VM's virtual disk.

The question: should application config directories (Jellyfin, Radarr, etc.) also live
on the SSD passthrough, alongside the media files?

---

## Decision

Application config stays on the VM's virtual disk. Only raw media files live on the SSD.

---

## Reason

During early testing, the SSD was disconnected from the VM under write load
(<!-- INPUT: describe what caused the disconnect — hot unplug, cable issue, etc. -->).

The result: Jellyfin's config database was mid-write when the device disappeared.
The config was corrupted. Recovery required restoring from a Proxmox snapshot.

**The lesson:** Passthrough devices are not as reliable as virtual disks under Proxmox.
They can disappear without graceful unmount if the physical connection is interrupted.
Named Docker volumes backed by the VM disk survive this scenario — the VM disk is
managed by Proxmox and benefits from QEMU's I/O handling.

Media files (read-mostly, append-only, easily re-downloaded) can tolerate the risk
of passthrough. Config files (random writes, painful to reconstruct) cannot.

---

## Consequences

- Jellyfin config is small and fast on the VM disk — no performance concern
- SSD usage is purely for media: read-heavy, tolerant of abrupt disconnects
- Recovery from SSD failure does not require restoring config — the two failure domains are now independent
- Tradeoff: VM disk space must be sized to hold all service configs (currently well under 10GB)
