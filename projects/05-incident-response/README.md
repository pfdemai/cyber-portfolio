# 05 — Incident Response

Structured playbooks based on the NIST IR lifecycle.
Applied to realistic scenarios on this infrastructure.

---

## NIST IR Lifecycle

| Phase | What it means |
|-------|---------------|
| Preparation | Tools, runbooks, and baselines in place before an incident |
| Detection & Analysis | Identify that something is wrong and understand what |
| Containment | Stop the bleeding — limit further damage without destroying evidence |
| Eradication | Remove the root cause |
| Recovery | Restore normal operation |
| Post-Incident Review | Document what happened, what worked, what to improve |

---

## Playbooks

| Scenario | File |
|----------|------|
| Unknown process generating outbound traffic at 3am | [playbooks/unknown-outbound-traffic.md](playbooks/unknown-outbound-traffic.md) |
