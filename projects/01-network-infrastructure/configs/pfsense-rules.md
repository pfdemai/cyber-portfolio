# pfSense Firewall Rules

Sanitized representation of active rules. Real IPs and hostnames replaced with
descriptive labels. Rules are documented in evaluation order per interface.

---

## WAN Interface

| # | Action | Protocol | Source | Destination | Port | Reason |
|---|--------|----------|--------|-------------|------|--------|
| 1 | Block | Any | Any | Any | Any | Default deny — WAN is untrusted |

No inbound rules on WAN. Remote access is handled entirely by Tailscale (UDP 41641 outbound from the LXC, not an inbound rule).

---

## LAN Interface (vmbr0)

| # | Action | Protocol | Source | Destination | Port | Reason |
|---|--------|----------|--------|-------------|------|--------|
| 1 | Pass | Any | LAN net | Any | Any | LAN hosts have outbound internet access |
| 2 | Block | Any | LAN net | LAB net | Any | LAN cannot initiate connections to lab zone |

<!-- INPUT: Add your actual LAN rules here. Document the reason for each rule. -->

---

## LAB Interface (vmbr1)

| # | Action | Protocol | Source | Destination | Port | Reason |
|---|--------|----------|--------|-------------|------|--------|
| 1 | Pass | TCP/UDP | LAB net | Any | 80, 443 | HTTP/HTTPS outbound for package updates and indexers |
| 2 | Pass | UDP | Tailscale LXC | Any | 41641 | Tailscale WireGuard tunnel |
| 3 | Block | Any | LAB net | LAN net | Any | Lab cannot reach LAN — enforces isolation |
| 4 | Block | Any | Any | Any | Any | Default deny |

<!-- INPUT: Adjust port list and source IPs to match your actual ruleset. -->

---

## Design Decisions

**No inbound WAN rules:** Tailscale eliminates the need to expose any port on the WAN. The firewall has zero inbound attack surface from the internet.

**LAB → LAN blocked at firewall:** Even if a service on vmbr1 is compromised, it cannot pivot to LAN. The block is enforced at pfSense, not just by routing.

**Stateful inspection:** pfSense tracks connection state. Return traffic for established LAN→WAN connections is allowed automatically without an explicit inbound rule.
