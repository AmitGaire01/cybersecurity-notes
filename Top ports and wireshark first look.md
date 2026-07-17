# TCP/UDP, Top Ports & Wireshark First Look
**Date:** June 14, 2026  
**Goal:** Understand transport layer protocols, memorize key ports, capture first live traffic

---

## 1. TCP vs UDP — Deep Comparison

| Feature        | TCP                              | UDP                            |
|----------------|----------------------------------|--------------------------------|
| Full Name      | Transmission Control Protocol    | User Datagram Protocol         |
| Connection     | Connection-oriented              | Connectionless                 |
| Reliability    | Guaranteed — resends lost data   | No guarantee — fire and forget |
| Order          | Packets arrive in order          | No ordering                    |
| Speed          | Slower (overhead)                | Faster (no handshake)          |
| Error checking | Yes                              | Minimal                        |
| Use cases      | HTTP, SSH, FTP, SMTP, RDP        | DNS, VoIP, video streaming     |

---


```bash
## 2. TCP 3-Way Handshake
Client ──── SYN ────────────────► Server
Client ◄─── SYN-ACK ────────────  Server
Client ──── ACK ────────────────► Server
✅ Connection Established

- **SYN** — Client says "I want to connect, here's my sequence number"
- **SYN-ACK** — Server says "OK, I acknowledge, here's mine"
- **ACK** — Client confirms, connection is open

### TCP 4-Way Connection Termination
Client ──── FIN ────────────────► Server
Client ◄─── ACK ─────────────── Server
Client ◄─── FIN ─────────────── Server
Client ──── ACK ────────────────► Server
✅ Connection Closed
```
> SOC relevance: A massive flood of SYN packets with no ACK response
> = SYN Flood DDoS attack. Look for this in firewall and IDS logs.

---

## 3. Top Ports to Memorize

| Port  | Protocol | Service              | Security Note                        |
|-------|----------|----------------------|--------------------------------------|
| 21    | TCP      | FTP                  | Sends credentials in plaintext ⚠️    |
| 22    | TCP      | SSH                  | Secure remote login                  |
| 23    | TCP      | Telnet               | Plaintext — should never be open ⚠️  |
| 25    | TCP      | SMTP                 | Email sending                        |
| 53    | TCP/UDP  | DNS                  | Abused for C2, exfiltration ⚠️       |
| 80    | TCP      | HTTP                 | Unencrypted web traffic ⚠️           |
| 110   | TCP      | POP3                 | Email retrieval (older)              |
| 143   | TCP      | IMAP                 | Email retrieval (modern)             |
| 443   | TCP      | HTTPS                | Encrypted web traffic ✅             |
| 445   | TCP      | SMB                  | File sharing — EternalBlue vuln ⚠️   |
| 3306  | TCP      | MySQL                | Database — should not be public ⚠️   |
| 3389  | TCP      | RDP                  | Remote Desktop — brute forced often ⚠️|
| 8080  | TCP      | HTTP-alt             | Web proxy, dev servers               |
| 8443  | TCP      | HTTPS-alt            | Alternate HTTPS                      |

> SOC tip: If you see ports 23, 445, or 3389 open on an internet-facing
> host during threat hunting — that's an immediate red flag to investigate.

---

## 4. Wireshark — First Capture

### What is Wireshark?
A network protocol analyzer (packet sniffer) that captures and inspects
every packet flowing through your network interface in real time.
Used daily by SOC analysts for network traffic analysis.

### Key Display Filters Used Today

| Filter              | What it shows                        |
|---------------------|--------------------------------------|
| `tcp`               | All TCP traffic only                 |
| `udp`               | All UDP traffic only                 |
| `dns`               | DNS queries and responses            |
| `http`              | Unencrypted HTTP traffic             |
| `ip.addr == x.x.x.x` | Traffic to/from a specific IP      |
| `tcp.port == 443`   | HTTPS traffic only                   |

### OSI Layers in Wireshark
Every packet in Wireshark shows all layers:
- Frame (Physical — Layer 1)
- Ethernet (Data Link — Layer 2)
- Internet Protocol (Network — Layer 3)
- TCP/UDP (Transport — Layer 4)
- HTTP/DNS etc. (Application — Layer 7)

This is the OSI model from Day 1 visible in real packets!

### What I Captured Today
- Captured live traffic on WiFi adapter for 3 minutes
- Applied tcp, udp, dns, http filters
- Found a DNS query packet — domain being resolved: [write what you saw]
- Identified TCP handshake packets (SYN, SYN-ACK, ACK)

---

## 5. Key Takeaways for SOC Work
- TCP is reliable but slower — used for anything that needs delivery guarantee
- UDP is fast but unreliable — DNS uses UDP (port 53) for speed
- Open ports tell a story — a SOC analyst reads open ports like a fingerprint
- Wireshark lets you see exactly what's on the wire — nothing is hidden
- DNS traffic in Wireshark is goldmine for SOC — shows every domain queried

---

## TryHackMe
- [x] Introductory Networking room — completed

## Commands / Tools Used Today
- Wireshark — live capture on Windows
- Display filters: tcp, udp, dns, http
