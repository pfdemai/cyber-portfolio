# Playbook: Unknown Outbound Traffic from Media VM

**Scenario:** At 03:00, pfSense logs show sustained outbound TCP connections from the
media VM to an unfamiliar external IP on port 443. The traffic pattern is regular —
one connection every 30 seconds. No service on the media VM should be generating
scheduled outbound traffic at this time.

**Assumption:** This is a constructed scenario on the homelab infrastructure.
All actions below use real tools available in this environment.

---

## Phase 1 — Detection & Analysis

**Objective:** Confirm the anomaly is real, identify the source process, assess severity.

**Step 1.1 — Verify the pfSense alert**

Log into pfSense UI → Status → System Logs → Firewall.
Filter by source IP (media VM's vmbr1 address).

```bash
# Alternatively, SSH to pfSense and query logs directly
clog /var/log/filter.log | grep <media-vm-ip>
```

Expected: repeated outbound connections to the same external IP, consistent interval.

**Step 1.2 — Check DNS resolution for the destination IP**

```bash
dig -x <destination-ip>
# or
whois <destination-ip>
```

A residential IP or a known C2 hosting range (e.g., DigitalOcean, Vultr with no legitimate service) raises severity. A known CDN or update service lowers it.

**Step 1.3 — Identify the source process on the media VM**

SSH to the media VM via Tailscale.

```bash
# Show active connections with process info
ss -tnp | grep <destination-ip>

# If ss doesn't show the process (permission issue):
sudo netstat -tnp | grep <destination-ip>
```

Note the PID and process name.

```bash
# Get full process details
ps aux | grep <PID>

# Show process binary and open files
ls -la /proc/<PID>/exe
lsof -p <PID>
```

**Step 1.4 — Check if it's a container or host process**

```bash
# If it's a Docker container process, identify the container
docker ps -q | xargs docker inspect --format '{{.State.Pid}} {{.Name}}' | grep <PID>
```

**Step 1.5 — Assess severity**

| Finding | Severity |
|---------|---------|
| Known process (qBittorrent peer, update check) | Low — document and close |
| Unknown binary in /tmp or /dev/shm | Critical — move to containment |
| Process spawned by a web server or media service | High — likely exploit, move to containment |
| Container process with no network rule justification | High — move to containment |

---

## Phase 2 — Containment

**Objective:** Stop further data exfiltration without destroying forensic evidence.

**Step 2.1 — Isolate at the firewall (preferred over killing the process)**

In pfSense, add a temporary block rule on the LAB interface:

```
Action: Block
Protocol: Any
Source: <media-vm-ip>
Destination: <destination-ip>
Description: INCIDENT CONTAINMENT - unknown outbound traffic
```

This stops the traffic while leaving the process running and the system state intact.

**Step 2.2 — Capture a memory dump before any further action**

```bash
# On the media VM — requires LiME or equivalent
# If LiME is not available, at minimum capture process memory
sudo gcore -o /tmp/incident-$(date +%Y%m%d%H%M) <PID>
```

**Step 2.3 — Preserve logs**

```bash
# Snapshot Docker logs
docker logs <container-name> > /tmp/incident-logs-$(date +%Y%m%d).txt 2>&1

# Preserve system journal
journalctl -u <relevant-service> --since "2 hours ago" > /tmp/journal-dump.txt
```

**Step 2.4 — Take a Proxmox snapshot of the VM**

In the Proxmox UI: VM → Snapshots → Take Snapshot.
Name it `incident-YYYYMMDD-pre-remediation`. This preserves disk state.

---

## Phase 3 — Eradication

**Objective:** Remove the root cause.

**Step 3.1 — Identify how the process got there**

```bash
# Check recently modified files
find / -newer /tmp/reference-timestamp -type f 2>/dev/null | grep -v proc | grep -v sys

# Check crontabs
crontab -l
cat /etc/cron.d/*

# Check persistence mechanisms
ls -la ~/.bashrc ~/.profile /etc/rc.local
```

**Step 3.2 — Remove the malicious artifact**

```bash
# Kill the process
kill -9 <PID>

# Remove the binary
rm -f <path-to-binary>

# Remove any persistence (cron entry, systemd unit, etc.)
```

**Step 3.3 — Rotate any credentials the compromised service had access to**

Check the `.env` file — rotate API keys for Radarr, Sonarr, Prowlarr if the container was compromised.

---

## Phase 4 — Recovery

**Step 4.1 — Verify the outbound traffic has stopped**

Monitor pfSense logs for 15 minutes. Confirm no further connections to the destination IP.

**Step 4.2 — Restore the service if it was taken offline**

```bash
docker compose up -d <service-name>
```

**Step 4.3 — Remove the containment firewall rule**

Delete the temporary block rule added in Phase 2. Confirm normal traffic resumes.

---

## Phase 5 — Post-Incident Review

Document within 24 hours:

- **Timeline:** When was the traffic first generated? When detected? How long between detection and containment?
- **Root cause:** How did the process get on the system?
- **Detection gap:** Would this have been detected faster with a SIEM? What rule would have caught it?
- **Action items:**
  - [ ] Deploy Wazuh to centralize logs and enable real-time alerting
  - [ ] Establish network baseline (expected outbound destinations per service)
  - [ ] Add FIM to detect unexpected file writes in container volumes

---

## Notes on This Playbook

This scenario was constructed to demonstrate IR methodology. The steps use real tools
available in this environment. The specific incident did not occur — but every command
is executable, and every decision is defensible.

In a real SOC, containment authority and escalation paths would be defined before the incident.
The Preparation phase is what makes the rest of the lifecycle executable under pressure.
