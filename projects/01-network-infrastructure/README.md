# 01 — Network Infrastructure & Segmentation

Self-hosted homelab built on Proxmox, segmented with pfSense, accessed remotely via Tailscale.
This document explains the architecture decisions — not just the topology, but why it was designed this way.

---

## Infrastructure Overview

- **Hypervisor:** Proxmox VE on a MacBook 2015 (repurposed hardware)
- **Firewall/Router:** pfSense — single enforcement point for all inter-zone traffic
- **Remote access:** Tailscale (WireGuard-based mesh VPN, deployed as a subnet router in an LXC)
- **Network bridges:** vmbr0 (LAN), vmbr1 (isolated lab network)

See [diagrams/network-topology.md](diagrams/network-topology.md) for the full topology with trust zone mapping.

---

## Segmentation Rationale

Two bridges serve two purposes.

**vmbr0 (LAN):** Management traffic. The Proxmox host interface, administration access.
High trust — only known devices.

**vmbr1 (Lab):** All VMs and containers that run services. Medium trust — isolated from LAN.
A compromise here does not grant LAN access. pfSense enforces this at the firewall level,
not just by routing.

This maps directly to **defense in depth**: the media stack is exposed to the internet
(indexers, torrent client), so it runs in the least-trusted zone. Even if a container
is compromised, lateral movement is blocked at the network layer.

---

## Remote Access Design

Tailscale runs as a subnet router in an LXC on vmbr1. It advertises the vmbr1 subnet
to the Tailscale mesh, allowing remote access to all lab VMs without opening any WAN port.

Zero inbound rules on the pfSense WAN interface. The entire remote access surface is
the Tailscale WireGuard tunnel — authenticated, encrypted, invisible to port scanners.

---

## Firewall Rules

See [configs/pfsense-rules.md](configs/pfsense-rules.md) for the sanitized ruleset
with per-rule explanations.

---

## Incident Record

**[IP Conflict — vmbr1 & Tailscale LXC](incidents/ip-conflict-vmbr1-tailscale.md)**
A silent IP conflict broke remote access. Symptom → diagnosis → resolution documented.
Demonstrates methodical troubleshooting at the network layer.

---

## Skills Demonstrated

- Network segmentation and trust zone design
- pfSense firewall administration (stateful ruleset, DHCP reservation)
- Tailscale deployment as subnet router
- Layer 2 troubleshooting (ARP conflict diagnosis)
- Architecture decisions backed by security rationale
