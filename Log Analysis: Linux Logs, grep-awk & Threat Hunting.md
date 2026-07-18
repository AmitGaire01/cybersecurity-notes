# Day 12 — Log Analysis: Linux Logs, grep/awk & Threat Hunting
**Date:** June 24, 2026  
**Goal:** Hunt threats in raw log files like a SOC analyst

---

## 1. Why Log Analysis Matters

Logs are the single most important data source for a SOC analyst.
Every login, process, network connection, and error is recorded in logs.
When an attack happens, logs tell you:
- When it happened
- Where it came from
- What the attacker did
- Which systems were affected

> SOC tip: "Logs don't lie" — even if an attacker covers their tracks,
> logs on network devices, firewalls, and SIEMs usually capture evidence.

---

## 2. Linux Log Files — /var/log

| Log File | Contains | SOC Use |
|---|---|---|
| `/var/log/auth.log` | Authentication events — logins, sudo, SSH | Brute force, unauthorized access |
| `/var/log/syslog` | General system events | System changes, service starts/stops |
| `/var/log/kern.log` | Kernel messages | Hardware issues, kernel exploits |
| `/var/log/apache2/access.log` | Web server requests | Web attacks, scanning, SQLi |
| `/var/log/apache2/error.log` | Web server errors | Application errors, attack attempts |
| `/var/log/nginx/access.log` | Nginx web requests | Same as Apache access log |
| `/var/log/dpkg.log` | Package installations/removals | Unauthorized software installed |
| `/var/log/cron.log` | Scheduled task executions | Malicious cron jobs (persistence) |
| `/var/log/faillog` | Failed login attempts | Brute force detection |
| `/var/log/wtmp` | Login history (use `last` command) | Who logged in and when |
| `/var/log/btmp` | Failed login history (use `lastb`) | Failed login history |

---

## 3. Essential Log Analysis Commands

```bash
# Display full log file
cat /var/log/auth.log

# Show last 20 lines
tail -20 /var/log/auth.log

# Follow log in real time (live monitoring)
tail -f /var/log/auth.log

# Show first 20 lines
head -20 /var/log/auth.log

# Scroll through log (q to quit)
less /var/log/auth.log

# Show login history
last

# Show failed login history
lastb

# Find all failed SSH logins
grep "Failed password" /var/log/auth.log

# Find successful logins
grep "Accepted password" /var/log/auth.log

# Find sudo usage
grep "sudo" /var/log/auth.log

# Case insensitive search
grep -i "error" /var/log/syslog

# Show line numbers
grep -n "Failed" /var/log/auth.log

# Count matching lines
grep -c "Failed password" /var/log/auth.log

# Print field 11 (IP address in failed logins)
awk '{print $11}' /var/log/auth.log

# Print fields 1,2,3 (date and time)
awk '{print $1, $2, $3}' /var/log/auth.log

# Sort and count occurrences
sort /var/log/auth.log | uniq -c | sort -rn
```

---

## 4. Powerful Log Analysis Pipelines

```bash
# TOP ATTACKING IPs — brute force detection
grep "Failed password" /var/log/auth.log \
  | awk '{print $11}' \
  | sort \
  | uniq -c \
  | sort -rn \
  | head -10

# Find all usernames being brute forced
grep "Failed password" /var/log/auth.log \
  | awk '{print $9}' \
  | sort \
  | uniq -c \
  | sort -rn

# Check if brute force IP also had successful login (compromised!)
grep "Failed password" /var/log/auth.log | grep "185.220.101.45"
grep "Accepted password" /var/log/auth.log | grep "185.220.101.45"

# Find 404 errors in Apache (scanning)
grep " 404 " /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# Find SQL injection attempts
grep -i "union\|select\|drop\|insert" /var/log/apache2/access.log

# Find directory traversal attempts
grep "\.\./\.\." /var/log/apache2/access.log

# Find cron jobs added (persistence)
grep "CRON" /var/log/syslog

# Find sudo commands run (privilege escalation)
grep "sudo" /var/log/auth.log | grep "COMMAND"
```

---

## 5. Auth.log — Sample Log Analysis

```
# Sample auth.log entries:

Jun 24 10:23:45 server sshd[1234]: Failed password for root from 185.220.101.45 port 52341 ssh2
Jun 24 10:23:46 server sshd[1234]: Failed password for root from 185.220.101.45 port 52342 ssh2
Jun 24 10:23:47 server sshd[1234]: Failed password for admin from 185.220.101.45 port 52343 ssh2
Jun 24 10:24:01 server sshd[1235]: Accepted password for amit from 192.168.1.5 port 54231 ssh2
Jun 24 10:24:05 server sudo[1236]: amit : TTY=pts/0 ; PWD=/home/amit ; USER=root ; COMMAND=/bin/bash

# Field breakdown:
# Jun 24 10:23:45     → Timestamp
# server              → Hostname
# sshd[1234]          → Process name and PID
# Failed password     → Event type
# for root            → Target username
# from 185.220.101.45 → Source IP ← ATTACKER
# port 52341          → Source port

# Red flags:
# 3 failed logins in 2 seconds    = brute force attack
# Targeting "root" and "admin"    = credential stuffing
# All from same IP                = single attacker
# sudo COMMAND=/bin/bash          = root shell obtained
```

---

## 6. Apache Access Log — Sample Analysis

```
# Sample access.log entry:
192.168.1.100 - - [24/Jun/2026:10:30:00 +0545] "GET /admin/config.php HTTP/1.1" 404 512 "-" "sqlmap/1.6"

# Field breakdown:
# 192.168.1.100   → Client IP
# [24/Jun/2026]   → Timestamp (+0545 = Nepal timezone!)
# GET             → HTTP method
# /admin/config.php → Requested path
# 404             → HTTP status (not found)
# 512             → Response size bytes
# sqlmap/1.6      → User agent ← ATTACK TOOL IDENTIFIED

# Red flags:
# Requesting /admin/ paths     = admin panel hunting
# 404 responses                = scanning for files
# sqlmap user agent            = automated SQL injection tool
```

---

## 7. Threat Indicators in Logs

| What you see | What it means |
|---|---|
| 100+ failed logins in 60 seconds | Brute force attack |
| Failed logins for many usernames from one IP | Credential stuffing |
| Successful login after many failures | Account compromised |
| Login at 3am from foreign IP | Suspicious — investigate |
| sudo COMMAND=/bin/bash | Root shell obtained |
| New cron job added | Persistence mechanism |
| sqlmap/nikto/nmap in user agent | Automated attack tool |
| ../ in web requests | Directory traversal attempt |
| union+select in web requests | SQL injection attempt |
| Large outbound data transfer | Possible data exfiltration |

---

## 8. Key Takeaways for SOC Work

- auth.log = first file to check in any Linux security incident
- `grep "Failed password" | awk '{print $11}' | sort | uniq -c | sort -rn` = instant brute force IP list
- Successful login after failures = account breach until proven otherwise
- Apache logs reveal attack tools via user agent field
- `tail -f` = live log monitoring during active incidents
- Always check timestamps — attacks at odd hours = red flag
- Nepal timezone is +0545 — keep this in mind when reading logs

---

## TryHackMe
- [x] Advent of Cyber 2023 Day 3 (log analysis) — completed

## Commands Practiced Today

```bash
grep "Failed password" /var/log/auth.log
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
grep "Accepted password" /var/log/auth.log
tail -f /var/log/auth.log
last
lastb
grep -i "union\|select" /var/log/apache2/access.log
```


