# purple-writeups

Attack walkthroughs paired with the detection logic that catches them.

Each entry follows the same loop: **execute → observe telemetry → confirm
the detection fires → note how to improve both sides.**

## Write-ups

| # | Title | Attack | Detection | ATT&CK |
|---|-------|--------|-----------|--------|
| 001 | [Port scan with netrecon](writeups/001-netrecon-portscan/) | `netrecon` | `port_scan_burst` | T1046 |

## Why this exists
Running a tool proves you can follow steps. Pairing the attack with its
detection proves you understand the system from both sides — which is what
purple-team and detection-engineering roles actually require.
