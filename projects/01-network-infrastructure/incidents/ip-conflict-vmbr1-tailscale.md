# Incident: Silent IP Conflict — vmbr1 & Tailscale LXC

**Date:** <!-- INPUT: actual date -->
**Duration:** <!-- INPUT: time from symptom to resolution -->
**Impact:** Remote access via Tailscale fully broken

---

## Symptom

Tailscale reported the node as online but SSH connections to the media VM timed out.
Pinging the Tailscale IP (100.x.x.x) succeeded from a remote machine — ICMP replies came back —
but TCP connections to any port on the media VM failed silently.

---

## Hypotheses

Before touching anything, three candidates:

1. **Tailscale daemon issue** — process crashed or lost its tunnel state
2. **Firewall rule regression** — a pfSense rule change had blocked the return path
3. **IP conflict on vmbr1** — another host on the lab bridge had taken the Tailscale LXC's IP

---

## Diagnosis

**Check 1 — Tailscale daemon on the LXC:**
```bash
systemctl status tailscaled
# Result: active (running) — daemon is fine
tailscale status
# Result: showed the node as connected, no errors
```

**Check 2 — pfSense logs:**
Firewall logs showed no blocked traffic from the Tailscale LXC IP during the incident window.
Rule regression ruled out.

**Check 3 — ARP table on the Proxmox host:**
```bash
arp -n | grep <vmbr1-subnet>
# Result: two entries with the same IP — one for the Tailscale LXC MAC, one for another VM
```

Root cause confirmed: a newly started VM on vmbr1 had been assigned <!-- INPUT: the conflicting IP --> via DHCP, and it matched the static IP configured on the Tailscale LXC. ARP responses were split between the two MACs. TCP sessions established against the wrong host, which had no listener — silent timeout.

---

## Resolution

1. Identified the conflicting VM and shut it down
2. Assigned it a static IP outside the DHCP range on vmbr1
3. Flushed the ARP cache on the Proxmox host: `ip neigh flush dev vmbr1`
4. Tailscale connectivity restored immediately

**Permanent fix:** Reserved the Tailscale LXC's IP in the pfSense DHCP server (MAC binding) to prevent re-occurrence. The LXC's static IP now sits outside the DHCP pool.

---

## What This Demonstrates

The failure was silent because ICMP (ping) arrived at the correct host via an ARP race while TCP sessions consistently hit the wrong one. The diagnostic process mattered: three hypotheses, eliminate each with a specific check, confirm with ARP inspection. No panic, no reboots.

**Takeaway for a SOC context:** Silent failures that appear as connectivity issues often resolve to layer 2 — ARP, MAC tables, VLAN tagging. Check the physical/link layer before blaming the application or firewall.
