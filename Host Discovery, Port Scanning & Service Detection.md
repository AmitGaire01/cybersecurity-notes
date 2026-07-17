# Nmap: Host Discovery, Port Scanning & Service Detection
**Date:** June 20, 2026  
**Goal:** Master Nmap — the #1 network scanning tool used by SOC analysts and pentesters

---

## 1. What is Nmap?

Nmap (Network Mapper) is a free, open-source network scanner used to:
- Discover live hosts on a network
- Find open ports on a target
- Detect running services and versions
- Identify the operating system
- Find potential vulnerabilities

Used by both attackers (reconnaissance) and defenders (network auditing).
As a SOC analyst, you use Nmap to audit your own network and verify
what's exposed to the internet.

> Legal reminder: Only scan networks and systems you own or have
> explicit written permission to scan. Unauthorized scanning is
> illegal under Nepal's Electronic Transaction Act.

---

## 2. Nmap Scan Types

| Scan Type        | Flag    | How it works                                         | When to use                    |
|------------------|---------|------------------------------------------------------|--------------------------------|
| SYN Scan         | -sS     | Sends SYN, gets SYN-ACK, sends RST (never completes) | Default, stealthy, fast        |
| TCP Connect      | -sT     | Full 3-way handshake — logged by target              | When SYN scan not available    |
| UDP Scan         | -sU     | Sends UDP packets — slower, less reliable            | Find DNS, SNMP, DHCP services  |
| Ping Sweep       | -sn     | Just checks if host is alive, no port scan           | Network discovery              |
| Null Scan        | -sN     | No flags set — may bypass some firewalls             | Stealthy recon                 |
| FIN Scan         | -sF     | Sends FIN flag — may bypass some firewalls           | Stealthy recon                 |
| Version Scan     | -sV     | Detects service versions on open ports               | Service enumeration            |
| OS Detection     | -O      | Guesses OS based on TCP/IP fingerprint               | OS fingerprinting              |

---

## 3. Essential Nmap Flags

| Flag          | Meaning                                              | Example                          |
|---------------|------------------------------------------------------|----------------------------------|
| `-sV`         | Detect service/version on open ports                 | `nmap -sV target`                |
| `-sC`         | Run default scripts (NSE)                            | `nmap -sC target`                |
| `-A`          | Aggressive: OS detect + version + scripts + traceroute | `nmap -A target`               |
| `-p-`         | Scan all 65535 ports                                 | `nmap -p- target`                |
| `-p 80,443`   | Scan specific ports only                             | `nmap -p 80,443 target`          |
| `-Pn`         | Skip ping — treat host as alive (useful if ICMP blocked) | `nmap -Pn target`            |
| `-O`          | OS detection                                         | `nmap -O target`                 |
| `-T0` to `-T5`| Speed — T0 slowest/stealthy, T4 fast, T5 aggressive | `nmap -T4 target`                |
| `-sn`         | Ping sweep — no port scan, just find live hosts      | `nmap -sn 192.168.1.0/24`        |
| `-oN file`    | Save output to normal text file                      | `nmap -oN results.txt target`    |
| `-oX file`    | Save output to XML file                              | `nmap -oX results.xml target`    |
| `-v`          | Verbose — show more detail during scan               | `nmap -v target`                 |
| `--script`    | Run specific NSE script                              | `nmap --script vuln target`      |

---

## 4. Most Used Nmap Commands

### Basic scan — top 1000 ports
```bash
nmap scanme.nmap.org
```

### Service and version detection
```bash
nmap -sV scanme.nmap.org
```

### Aggressive scan — everything at once
```bash
nmap -A -T4 scanme.nmap.org
```

### Scan all 65535 ports
```bash
nmap -p- scanme.nmap.org
```

### Ping sweep — find all live hosts on network
```bash
nmap -sn 192.168.1.0/24
```

### Scan specific ports
```bash
nmap -p 22,80,443,3389 scanme.nmap.org
```

### Save results to file
```bash
nmap -A -T4 -oN scan_results.txt scanme.nmap.org
```

### Run vulnerability scripts
```bash
nmap --script vuln scanme.nmap.org
```

---

## 5. Reading Nmap Output

```bash
Starting Nmap 7.94 at 2026-06-20
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.17s latency).
PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 6.6.1p1
80/tcp    open     http       Apache httpd 2.4.7
9929/tcp  open     nping-echo Nping echo
31337/tcp open     tcpwrapped
OS details: Linux 3.x|4.x
```

### Port States
| State      | Meaning                                              |
|------------|------------------------------------------------------|
| open       | Port is accepting connections — service running      |
| closed     | Port accessible but no service listening             |
| filtered   | Firewall blocking — Nmap can't determine state       |
| open/filtered | Can't tell if open or filtered (UDP scans)        |

---

## 6. Nmap Scripting Engine (NSE)

NSE scripts extend Nmap's functionality for deeper analysis.

| Script Category | Flag               | What it does                          |
|-----------------|--------------------|---------------------------------------|
| Default         | `-sC`              | Safe, commonly useful scripts         |
| Vulnerability   | `--script vuln`    | Check for known vulnerabilities       |
| Auth            | `--script auth`    | Test authentication                   |
| Brute           | `--script brute`   | Brute force login attempts            |
| Discovery       | `--script discovery` | Gather more info about target       |

### Example — check for SMB vulnerabilities (EternalBlue)
```bash
nmap --script smb-vuln-ms17-010 -p 445 target
```

---

<!-- ## 7. My Scan Results — scanme.nmap.org

### Command run:
```bash
nmap -A -T4 scanme.nmap.org
```

### Results:

-->
---

## 8. SOC Use Cases for Nmap

| Scenario                              | Nmap Command                              |
|---------------------------------------|-------------------------------------------|
| Find all live hosts on network        | `nmap -sn 192.168.1.0/24`                |
| Audit open ports on a server          | `nmap -p- -sV server_ip`                 |
| Check if RDP is exposed               | `nmap -p 3389 server_ip`                 |
| Verify firewall is blocking ports     | `nmap -Pn -p 22,23,445 server_ip`        |
| Check for EternalBlue vulnerability   | `nmap --script smb-vuln-ms17-010 -p 445` |

---

## 9. Key Takeaways for SOC Work
- `-A -T4` is your go-to scan for most situations
- Always save output with `-oN` — document everything
- `filtered` state = firewall present — good sign for defenders
- Port 445 open externally = check for EternalBlue immediately
- Nmap is loud — leaves traces in logs — use carefully in production
- scanme.nmap.org = only public legal target to practice on

---

## TryHackMe
- [x] Nmap room — completed

## Commands Practiced Today
```bash
nmap scanme.nmap.org
nmap -sV scanme.nmap.org
nmap -A -T4 scanme.nmap.org
nmap -p- scanme.nmap.org
nmap -sn 192.168.1.0/24
nmap -p 22,80,443 scanme.nmap.org
```
