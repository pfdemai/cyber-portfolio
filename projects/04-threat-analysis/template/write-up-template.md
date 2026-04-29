# [Attack Name / Scenario Title]

**Date:** YYYY-MM-DD
**Source:** [CTF name / Lab platform / Bootcamp exercise]
**MITRE ATT&CK:** [Tactic — Technique ID](https://attack.mitre.org/techniques/TXXX/)

---

## Scenario

Brief description of the context. What environment, what was the starting point,
what was the objective.

---

## Technique

How the attack works at a technical level. What it exploits, what preconditions are needed.
Keep this to 3-5 sentences — enough to explain it clearly in an interview.

---

## Approach

Step-by-step: what you did, what tools you used, what you observed.

```
# Example command
tool --flag target
# Output:
[relevant output here]
```

---

## Findings

What the attack achieved. What data was accessed, what privilege was gained,
what the attacker's position is post-exploitation.

---

## Detection Logic

**What the logs look like:**

```
[example log line showing the attack signature]
```

**SIEM rule (conceptual):**

```
IF event_count(failed_auth, src_ip=X, window=60s) > 10 THEN alert(brute_force)
```

**Indicators of Compromise:**
- [IOC 1 — e.g., unusual process spawned by a web server]
- [IOC 2 — e.g., lateral movement to domain controller]

---

## Takeaway

One or two sentences on what this scenario taught you about detection or defense.
