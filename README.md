# Cybersecurity Portfolio

Hands-on cybersecurity projects built around a self-hosted homelab.
Targeting SOC N1/N2 analyst roles.

This repository is the technical backbone of a portfolio showcased at
[pfdemai.fr](https://pfdemai.fr) — the website is in French, the repo stays in English.

---

## Projects

| # | Project | What it demonstrates |
|---|---------|----------------------|
| 01 | [Network Infrastructure & Segmentation](projects/01-network-infrastructure/) | Network design, trust zones, methodical troubleshooting |
| 02 | [Hardened Service Stack](projects/02-hardened-service-stack/) | Service isolation, secrets handling, operational decisions |
| 03 | [SOC Fundamentals](projects/03-soc-fundamentals/) | Security theory anchored to real infrastructure |
| 04 | [Threat Analysis](projects/04-threat-analysis/) | Attack scenario write-ups, detection logic, ATT&CK mapping |
| 05 | [Incident Response](projects/05-incident-response/) | NIST IR lifecycle, structured playbooks |

---

## Infrastructure

- **Hypervisor:** Proxmox on MacBook 2015
- **Firewall:** pfSense (enforcement layer)
- **Remote access:** Tailscale (subnet router)
- **Services:** Docker Compose stack (Jellyfin, Radarr, Sonarr, Prowlarr, qBittorrent, NordVPN)
- **Secrets:** .env with restricted permissions → Infisical (next step)

---

## Context

Currently completing an 18-week cybersecurity bootcamp (Ironhack, EN).
Parallel preparation for RNCP37680 — Administrateur d'Infrastructures Sécurisées (Niveau 6).
