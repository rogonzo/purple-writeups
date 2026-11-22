# 001 — Port Scan with netrecon, and the Detection That Catches It

**Tools:** `netrecon` (offense) · Sigma rule `port_scan_burst` (defense)
**ATT&CK:** [T1046 — Network Service Discovery](https://attack.mitre.org/techniques/T1046/)
**Date:** 2026-11-22

---

## Objective
Run a port scan against a lab host with my own tool, then verify that the
matching detection rule fires on the resulting telemetry. This is the
purple-team loop: execute an attack, confirm the defender sees it.

## Environment
- **Attacker / scanner:** Kali Linux (`192.168.1.10`)
- **Target:** Ubuntu lab VM (`192.168.1.20`) with ufw logging enabled
- **Detection source:** ufw firewall logs

## Offense — the scan

```bash
python3 netrecon.py 192.168.1.20 -p 1-200 -o output/scan.json
