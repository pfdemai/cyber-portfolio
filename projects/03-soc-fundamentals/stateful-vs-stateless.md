# Stateful vs Stateless Firewall — What pfSense Actually Does

---

## Stateless Firewall

A stateless firewall evaluates each packet independently against a ruleset.
It has no memory of previous packets. To allow a TCP reply, you must write an
explicit rule permitting inbound traffic on the return port.

**Example:** You allow outbound TCP to port 443. The server sends a reply.
A stateless firewall evaluates the reply as a new packet — if there is no rule
permitting inbound TCP from port 443, the reply is dropped.

This forces administrators to write permissive inbound rules ("allow inbound TCP
from any source on ports > 1024") that are difficult to reason about and easy to exploit.

---

## Stateful Firewall

A stateful firewall tracks connection state in a state table. When an outbound
TCP SYN is allowed, the firewall records the tuple (src IP, src port, dst IP, dst port, state).
The reply (SYN-ACK, then ACK) is matched against this state table and allowed
automatically — no inbound rule required.

**pfSense is stateful.** The LAN→Any pass rule allows outbound connections.
Return traffic is permitted by state tracking, not by an inbound rule.

---

## In the Homelab

pfSense's WAN interface has no inbound pass rules. This is only possible because:
1. pfSense is stateful — return traffic for initiated connections is handled by the state table
2. Tailscale handles remote access — no inbound port is needed

If pfSense were stateless, maintaining no inbound WAN rules while allowing outbound
internet access would be impossible.

---

## Why It Matters in a SOC Context

State table exhaustion is a real attack vector. A SYN flood can fill the state table,
causing pfSense to drop new legitimate connections (DoS). Monitoring state table
utilization is part of network health monitoring. pfSense exposes this via its dashboard —
a baseline is worth establishing so anomalies are detectable.
