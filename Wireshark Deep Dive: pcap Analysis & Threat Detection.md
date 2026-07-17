# Wireshark Deep Dive: pcap Analysis & Threat Detection
**Date:** June 21, 2026  
**Goal:** Analyze network traffic like a SOC analyst using Wireshark

---

## 1. Wireshark Interface Overview
```bash
  Menu Bar + Toolbar + Filter Bar            │
├─────────────────────────────────────────────┤
│  Packet List Panel                          │
│  (shows all captured packets — top panel)   │
├─────────────────────────────────────────────┤
│  Packet Details Panel                       │
│  (OSI layers expanded — middle panel)       │
├─────────────────────────────────────────────┤
│  Packet Bytes Panel                         │
│  (raw hex + ASCII — bottom panel)           │
```

### Packet List Columns
| Column      | Meaning                                    |
|-------------|--------------------------------------------|
| No.         | Packet number in capture                   |
| Time        | Timestamp of packet                        |
| Source      | Sender IP address                          |
| Destination | Receiver IP address                        |
| Protocol    | Protocol used (TCP, UDP, DNS, HTTP etc.)   |
| Length      | Packet size in bytes                       |
| Info        | Summary of packet content                  |

---

## 2. Capture Filters vs Display Filters

| Feature         | Capture Filter                        | Display Filter                         |
|-----------------|---------------------------------------|----------------------------------------|
| When applied    | Before capture starts                 | After capture — on existing data       |
| Syntax          | BPF (Berkeley Packet Filter)          | Wireshark display filter syntax        |
| Purpose         | Limit what gets captured              | Filter what you see from captured data |
| Example         | `port 80`                             | `http`                                 |
| Flexibility     | Less flexible                         | Very flexible — most common            |

---

## 3. Essential Display Filters

### Protocol Filters
| Filter              | Shows                                    |
|---------------------|------------------------------------------|
| `tcp`               | All TCP traffic                          |
| `udp`               | All UDP traffic                          |
| `http`              | HTTP traffic only                        |
| `dns`               | DNS queries and responses                |
| `icmp`              | Ping traffic                             |
| `ftp`               | FTP traffic                              |
| `ssh`               | SSH traffic                              |
| `arp`               | ARP requests and replies                 |

### IP and Port Filters
| Filter                        | Shows                                |
|-------------------------------|--------------------------------------|
| `ip.addr == 192.168.1.1`      | Traffic to OR from this IP           |
| `ip.src == 192.168.1.1`       | Traffic FROM this IP only            |
| `ip.dst == 192.168.1.1`       | Traffic TO this IP only              |
| `tcp.port == 443`             | HTTPS traffic                        |
| `tcp.port == 22`              | SSH traffic                          |
| `udp.port == 53`              | DNS traffic                          |
| `!(ip.addr == 192.168.1.1)`   | Exclude this IP                      |

### Content Filters
| Filter                              | Shows                              |
|-------------------------------------|------------------------------------|
| `http.request.method == "GET"`      | HTTP GET requests only             |
| `http.request.method == "POST"`     | HTTP POST requests only            |
| `http.response.code == 200`         | Successful HTTP responses          |
| `http.response.code == 404`         | Not found responses                |
| `dns.qry.name == "google.com"`      | DNS queries for google.com         |
| `frame contains "password"`         | Any packet containing "password"   |

### Combining Filters
| Operator | Example                                          |
|----------|--------------------------------------------------|
| `and`    | `ip.src == 192.168.1.1 and tcp.port == 80`       |
| `or`     | `tcp.port == 80 or tcp.port == 443`              |
| `not`    | `not arp`                                        |
| `==`     | `http.response.code == 404`                      |
| `contains` | `http.host contains "google"`                  |

---

## 4. Key Wireshark Features for SOC Analysis

### Follow TCP Stream
Right-click any TCP packet → Follow → TCP Stream
- Reconstructs entire conversation between two hosts
- Shows full HTTP requests and responses in readable form
- Can reveal usernames, passwords on unencrypted connections
- Essential for understanding what data was transferred

### Statistics Menu
| Feature                    | Location                        | What it shows                      |
|----------------------------|---------------------------------|------------------------------------|
| Protocol Hierarchy         | Statistics → Protocol Hierarchy | Breakdown of all protocols in pcap |
| Conversations              | Statistics → Conversations      | All IP pairs that communicated     |
| Endpoints                  | Statistics → Endpoints          | All unique IPs/MACs seen           |
| IO Graph                   | Statistics → IO Graph           | Traffic volume over time           |
| DNS Statistics             | Statistics → DNS                | All DNS queries and responses      |

### Export Objects
File → Export Objects → HTTP
- Extracts all files transferred over HTTP from pcap
- Can recover images, documents, executables downloaded
- Useful for malware analysis from network captures

---

## 5. Detecting Threats in Wireshark

### Port Scan Detection
```bash
Filter: tcp.flags.syn == 1 and tcp.flags.ack == 0
Look for: One source IP sending SYN to many different ports
Sign of: Nmap or other port scanner
```

### SYN Flood DDoS
```bash
Filter: tcp.flags.syn == 1 and tcp.flags.ack == 0
Look for: Massive volume of SYN packets, no completing handshakes
Sign of: SYN flood attack
```

### Cleartext Credentials
```bash
Filter: http and frame contains "password"
OR: ftp and frame contains "PASS"
Look for: Credentials sent over unencrypted protocols
Sign of: Security misconfiguration
```

### DNS Exfiltration
```bash
Filter: dns
Look for: Unusually long DNS queries, many subdomains of one domain
Example: aGVsbG8gd29ybGQ.evil-domain.com (base64 encoded data)
Sign of: Data being exfiltrated via DNS tunneling
```

### ARP Spoofing (MITM)
```bash
Filter: arp
Look for: Two different MACs claiming same IP address
Sign of: ARP spoofing / MITM attack in progress
```
---

## 6. pcap Analysis — Sample Capture Results

### File analyzed: http.cap (from Wireshark sample captures)

### What I found:
- Applied `http` filter — found [X] HTTP GET requests
- Used Follow TCP Stream — saw full HTTP conversation
- Applied `dns` filter — domains being resolved: [list what you saw]
- Statistics → Protocol Hierarchy showed: [paste your results]

---

## 7. Wireshark Color Coding

| Color          | Meaning                                    |
|----------------|--------------------------------------------|
| Green          | TCP traffic                                |
| Light blue     | UDP traffic                                |
| Dark blue      | DNS traffic                                |
| Black          | TCP errors, bad packets                    |
| Yellow/Orange  | ARP, routing protocols                     |
| Red            | Errors, dangerous traffic                  |

---

## 8. Key Takeaways for SOC Work
- Follow TCP Stream = see exactly what two hosts said to each other
- `frame contains "password"` = instant credential leak detector
- Protocol Hierarchy = first thing to run on unknown pcap
- Black packets in Wireshark = TCP errors worth investigating
- DNS filter + long subdomains = possible C2 or data exfiltration
- Export Objects = recover files attacker downloaded or uploaded

---

## TryHackMe
- [x] Wireshark: The Basics room — completed

## Tools Used Today
- Wireshark — pcap analysis
- Sample capture: http.cap from wiki.wireshark.org/SampleCaptures

## Filters Practiced Today
```bash
http
dns
tcp
ip.addr == x.x.x.x
http.request.method == "GET"
frame contains "password"
tcp.flags.syn == 1 and tcp.flags.ack == 0
```
