#  OSI Model, TCP/IP, DNS, HTTP/S
**Date:** June 13, 2026  
**Goal:** Understand foundational networking concepts used daily in SOC work

---

## 1. OSI Model (7 Layers)

| Layer | Name         | Protocol Examples     | What it does                          |
|-------|--------------|-----------------------|---------------------------------------|
| 7     | Application  | HTTP, DNS, FTP, SMTP  | Interface for user applications       |
| 6     | Presentation | SSL/TLS, JPEG, ASCII  | Encryption, compression, formatting   |
| 5     | Session      | NetBIOS, RPC          | Opens/maintains/closes sessions       |
| 4     | Transport    | TCP, UDP              | End-to-end delivery, ports            |
| 3     | Network      | IP, ICMP, ARP         | Logical addressing, routing           |
| 2     | Data Link    | Ethernet, MAC, Switch | Physical addressing, frames           |
| 1     | Physical     | Cables, Wi-Fi, Hubs   | Raw bits over physical medium         |

**Mnemonic (top → bottom):** All People Seem To Need Data Processing

---

## 2. TCP/IP Model (4 Layers)

| TCP/IP Layer    | Maps to OSI          | Protocols          |
|-----------------|----------------------|--------------------|
| Application     | Layers 5, 6, 7       | HTTP, DNS, FTP     |
| Transport       | Layer 4              | TCP, UDP           |
| Internet        | Layer 3              | IP, ICMP           |
| Network Access  | Layers 1, 2          | Ethernet, Wi-Fi    |

**Key difference:** OSI is theoretical (7 layers), TCP/IP is practical (4 layers).  
SOC analysts think in TCP/IP but reference OSI when troubleshooting.

---

## 3. TCP vs UDP

| Feature       | TCP                        | UDP                        |
|---------------|----------------------------|----------------------------|
| Connection    | Connection-oriented        | Connectionless             |
| Reliability   | Guaranteed delivery        | No guarantee               |
| Speed         | Slower                     | Faster                     |
| Use cases     | HTTP, SSH, FTP, email      | DNS, video streaming, VoIP |

### TCP 3-Way Handshake
```bash
Client → Server : SYN  (I want to connect)
Server → Client : SYN-ACK  (OK, I acknowledge)
Client → Server : ACK  (Connected!)
```
> SOC relevance: A flood of SYN packets with no ACK = SYN Flood DDoS attack

---

## 4. DNS — How Resolution Works

**Flow when you type google.com in browser:**
Browser → Local Cache → Recursive Resolver → Root DNS Server
→ TLD Server (.com) → Authoritative Name Server → Returns IP
→ Browser connects to IP

**Key DNS record types:**
| Record | Purpose                        |
|--------|--------------------------------|
| A      | Domain → IPv4 address          |
| AAAA   | Domain → IPv6 address          |
| MX     | Mail server for domain         |
| CNAME  | Alias for another domain       |
| TXT    | Verification, SPF records      |

**Commands practiced:**
```bash
nslookup google.com
nslookup nabilbank.com
```

> SOC relevance: DNS is abused for C2 (command & control), data exfiltration,  
> and phishing (typosquatting domains like g00gle.com)

---

## 5. HTTP vs HTTPS

| Feature    | HTTP               | HTTPS                        |
|------------|--------------------|------------------------------|
| Port       | 80                 | 443                          |
| Encryption | None               | TLS/SSL encrypted            |
| Security   | Data visible       | Data protected in transit    |

### Common HTTP Methods
| Method | Use                          |
|--------|------------------------------|
| GET    | Retrieve data from server    |
| POST   | Send data to server          |
| PUT    | Update existing resource     |
| DELETE | Delete a resource            |

### HTTP Status Codes
| Code | Meaning                  |
|------|--------------------------|
| 200  | OK — success             |
| 301  | Moved permanently        |
| 403  | Forbidden                |
| 404  | Not found                |
| 500  | Internal server error    |

> SOC relevance: A spike in 403s or 404s in web logs = possible scanning/attack
---
## 6. Key Takeaways for SOC Work
- OSI Layer 3 (Network) and Layer 4 (Transport) are where most attacks happen
- DNS queries are logged — suspicious domains = first IOC to investigate
- HTTPS doesn't mean safe — malware uses HTTPS too (encrypted C2 traffic)
- HTTP status codes in web server logs help detect attacks and scanning activity

---

## TryHackMe
- [x] Network Fundamentals room — completed

## Commands Run Today
```bash
nslookup google.com
nslookup nabilbank.com
tracert google.com        # Windows
```

