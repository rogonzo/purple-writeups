# 003 — Slow Scan Evasion, and the Detection That Catches It

**Tools:** `netrecon --delay` (offense) · Sigma rules `port_scan_burst` + `slow_scan` (defense)
**ATT&CK:** [T1046 — Network Service Discovery](https://attack.mitre.org/techniques/T1046/)
**Date:** 2026-11-25

---

## Objective
Take the port scan from write-up #001, add throttling to evade the existing
detection rule, confirm the rule now misses it, then write a second rule
that catches the slow scan.

This is the full loop: **build → detect → evade → re-detect.**

## Environment
- **Attacker:** Kali Linux (`192.168.1.10`)
- **Target:** Ubuntu lab VM (`192.168.1.20`) with ufw logging
- **Detection source:** ufw firewall logs

## Step 1 — Baseline: the fast scan is detected

```bash
python3 netrecon.py 192.168.1.20 -p 1-200 -o output/fast.json
The port_scan_burst rule (count(dst_port) by src_ip > 20 in 1m) fires.
Detected.

Step 2 — Evasion: throttle the scan
I added a --delay flag to netrecon that sleeps between port attempts:

bash
python3 netrecon.py 192.168.1.20 -p 1-200 -d 3 -o output/slow.json
At 3 seconds per port, 200 ports take ~10 minutes. In any rolling 1-minute
window, the scanner touches only ~20 ports — right at the threshold, and
mostly under it.

Result: port_scan_burst does not fire. Missed.

Step 3 — Re-detect: a slow-scan rule
New rule: detect-lab/rules/slow_scan.yml

Logic: count(dst_port) by src_ip > 10 over a 10-minute window. The slow
scan still touches 200 ports in 10 minutes — far above this threshold —
so it fires even though the per-minute rate is low.

Result: Detected.

Why this matters
A single rule is a snapshot. An attacker who sees the rule (or just paces
their scan) defeats it. The defense that survives is the one that tracks
total distinct ports over a longer window, not instantaneous rate.

What I'd improve
Attack side: randomize the delay (e.g. 1–6s) so the per-minute rate
is irregular — harder to threshold cleanly.

Detection side: combine both rules into one that alerts on either
high rate or high total distinct ports over 10m. Single rule, two patterns.

Add destination-port uniqueness to the count — a legitimate service
hitting the same port 500 times is not a scan; 200 distinct ports is.

Files
Offense: rogonzo/netrecon

Defense: rogonzo/detect-lab
