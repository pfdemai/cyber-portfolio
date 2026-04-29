# Log Sources & SIEM Data

What log sources currently exist in this infrastructure, what they contain,
and what detection use cases they would enable in a SIEM.

---

## Existing Log Sources

### pfSense Firewall Logs

**Format:** Syslog (RFC 5424)
**Content:** Blocked/passed packets with src IP, dst IP, protocol, port, interface, rule matched
**Detection use cases:**
- Port scan detection (high rate of blocks from single source)
- Policy violation (LAB→LAN traffic attempting to bypass the block rule)
- Unusual outbound destinations from the media VM

**Current state:** Logs are local to pfSense. Not forwarded to any collector.

---

### Docker Container Logs

**Format:** JSON (via Docker daemon)
**Content:** stdout/stderr of each container — application-level logs
**Detection use cases:**
- Jellyfin: authentication failures, unusual access patterns
- qBittorrent: unexpected peer connections, high upload volume
- Prowlarr/Radarr/Sonarr: API key exposure attempts, unusual query patterns

**Current state:** Accessible via `docker logs <container>`. Not centralized.

```bash
# View last 100 lines of Jellyfin logs
docker logs jellyfin --tail 100

# Follow qBittorrent logs in real time
docker logs qbittorrent -f
```

---

### Tailscale Logs

**Format:** JSON (via `journalctl -u tailscaled`)
**Content:** Node connections, authentication events, subnet routing activity
**Detection use cases:**
- Unexpected nodes joining the Tailscale network
- Authentication from unusual locations/devices
- Subnet route changes

**Current state:** Local to the Tailscale LXC via journald.

---

### Proxmox Host Logs

**Format:** Syslog + structured Proxmox API logs
**Content:** VM start/stop events, snapshot operations, user authentication to the web UI
**Detection use cases:**
- Unauthorized access to the Proxmox management interface
- Unexpected VM creation or deletion

**Current state:** Local to the Proxmox host at `/var/log/`.

---

## What a SIEM Would Enable

If these sources were forwarded to a SIEM (e.g., Wazuh), the following correlation rules become possible:

| Rule | Sources needed | What it detects |
|------|----------------|-----------------|
| Port scan from single IP | pfSense logs | High block rate from one source in a short window |
| LAB→LAN pivot attempt | pfSense logs | Any packet matching the LAB→LAN block rule |
| Container auth failure spike | Docker logs | >N Jellyfin login failures in X minutes |
| Tailscale node anomaly | Tailscale logs | New device connecting outside known device list |

**Current gap:** No SIEM ingestion is configured. This section documents the detection
capability that exists in theory — it will become operational when Wazuh is deployed.
