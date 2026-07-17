# Common Attacks: Phishing, MITM, DoS/DDoS
**Date:** June 17, 2026  
**Goal:** Understand the most common attacks SOC analysts deal with daily

---

## 1. Phishing Attacks

### What is Phishing?
A social engineering attack where an attacker tricks a victim into
revealing sensitive information (credentials, credit card numbers)
or installing malware by pretending to be a trusted entity.

### Types of Phishing

| Type           | Target          | Method                                          | Example                                    |
|----------------|-----------------|-------------------------------------------------|--------------------------------------------|
| Phishing       | Mass audience   | Bulk emails pretending to be banks, Netflix etc | "Your account is suspended — click here"   |
| Spear Phishing | Specific person | Personalized email using victim's real details  | Email to "Amit at Deerwalk" from fake HR   |
| Whaling        | C-level execs   | Targets CEO, CFO, directors                     | Fake legal notice to company CEO           |
| Vishing        | Anyone          | Voice call — fake bank/tech support             | "This is NIC Asia Bank, verify your OTP"  |
| Smishing       | Mobile users    | SMS with malicious link                         | Fake Daraz delivery SMS with link          |
| Clone Phishing | Email recipient | Exact copy of legit email with malicious link   | Duplicate of real NTC bill with fake link  |

### How to Identify a Phishing Email
- Sender domain doesn't match (support@nicasia-bank.com vs nicasia.com)
- Urgency language — "Act NOW", "Your account will be closed"
- Generic greeting — "Dear Customer" instead of your name
- Suspicious links — hover shows different URL than displayed
- Attachments — unexpected .exe, .zip, .doc with macros
- Poor grammar and spelling (not always present in targeted attacks)

### Phishing IOCs (Indicators of Compromise)
- Suspicious sender domain (typosquatting: arnazon.com, g00gle.com)
- Links to IP addresses instead of domain names
- Encoded URLs (bit.ly, tinyurl hiding destination)
- Email headers showing mismatched Reply-To address
- Attachments with double extensions (invoice.pdf.exe)

### How to Analyze an Email Header
Key fields to check:
``` bash
From:           → Who claims to have sent it
Reply-To:       → Where replies go (red flag if different from From)
Received:       → Actual path the email traveled (read bottom to top)
X-Originating-IP: → Real IP of sender
SPF/DKIM/DMARC: → Authentication checks — FAIL = suspicious
```
Tool: mxtoolbox.com/EmailHeaders.aspx — paste header, analyze instantly

---

## 2. Man-in-the-Middle (MITM) Attacks

### What is MITM?
An attacker secretly intercepts and potentially alters communication
between two parties who believe they are communicating directly.
``` bash 
Normal:   Alice ←————————————→ Bank Website
MITM:     Alice ←——→ Attacker ←——→ Bank Website
(reads/modifies everything)
```

### Types of MITM Attacks

| Attack          | How it works                                                    |
|-----------------|-----------------------------------------------------------------|
| ARP Spoofing    | Attacker sends fake ARP replies, associates their MAC with gateway IP |
| DNS Spoofing    | Attacker poisons DNS cache, redirects domain to malicious IP   |
| SSL Stripping   | Downgrades HTTPS to HTTP, removes encryption                   |
| WiFi Eavesdropping | Attacker creates fake hotspot (Evil Twin attack)            |
| Session Hijacking | Steals session cookie to impersonate logged-in user          |

### MITM IOCs
- Unexpected ARP table changes (arp -a shows duplicate MACs)
- SSL certificate warnings in browser
- Sudden HTTP instead of HTTPS on known secure sites
- Unusual network latency
- Unknown devices on network

### MITM Prevention
- Use HTTPS everywhere (check for padlock)
- VPN on public WiFi
- Enable HSTS (HTTP Strict Transport Security)
- Use DNSSEC
- Monitor ARP tables for anomalies

---

## 3. DoS and DDoS Attacks

### What is DoS?
Denial of Service — attacker overwhelms a target system with traffic
or requests, making it unavailable to legitimate users.

### DoS vs DDoS

| Feature       | DoS                          | DDoS                              |
|---------------|------------------------------|-----------------------------------|
| Source        | Single machine               | Multiple machines (botnet)        |
| Scale         | Limited                      | Massive — can hit Tbps            |
| Harder to block | No — just block one IP    | Yes — thousands of IPs            |
| Example       | Single attacker floods server| Mirai botnet took down Dyn DNS    |

### Types of DDoS Attacks

| Type              | Layer    | How it works                              | Example                    |
|-------------------|----------|-------------------------------------------|----------------------------|
| Volumetric        | Layer 3/4 | Flood bandwidth with traffic             | UDP Flood, ICMP Flood      |
| Protocol          | Layer 3/4 | Exhaust server/firewall resources        | SYN Flood, Ping of Death   |
| Application Layer | Layer 7  | Exhaust web server with HTTP requests    | HTTP Flood, Slowloris      |

### SYN Flood (Most Common)
```bash
Attacker sends thousands of SYN packets with fake source IPs
Server sends SYN-ACK and waits for ACK that never comes
Server's connection table fills up → legitimate users can't connect
```

### DDoS IOCs
- Sudden massive spike in traffic volume
- High number of incomplete TCP connections
- Traffic from unusual geographic locations
- Single type of packet flooding (all UDP, all SYN)
- Server response time degrades then becomes unavailable

### DDoS Mitigation
- Rate limiting — limit requests per IP
- Blackhole routing — drop all traffic to target IP
- CDN/Scrubbing centers — Cloudflare absorbs attack traffic
- Geo-blocking — block traffic from unexpected countries
- IDS/IPS rules to detect and drop flood traffic

---

## 4. Real-World Context for Nepal

### Common attacks seen in Nepal:
- Phishing via fake NTC, Ncell, NIC Asia SMS and emails
- Vishing — fake calls claiming to be from banks asking for OTP
- DDoS against government and banking websites
- Fake Daraz/Foodmandu order SMS with malicious links

### As a SOC Analyst your job is to:
1. Detect these attacks from logs and alerts
2. Triage — is this a real attack or false positive?
3. Contain — block malicious IPs, quarantine affected systems
4. Document — write incident report
5. Report — escalate to senior analyst or management

---

## 5. Key Takeaways
- Phishing is #1 attack vector — 90% of breaches start here
- Spear phishing is harder to detect — uses personal details
- MITM is invisible to victims — SSL/TLS is your best defense
- SYN Flood = most common DoS — look for incomplete TCP handshakes
- Every attack leaves IOCs — your job as SOC analyst is to find them

---

## TryHackMe
- [x] Phishing room — completed
- [x] Phishing Emails 1 room — completed

## Tools Used Today
- PhishTank — browsed real phishing URLs
- MXToolbox Email Header Analyzer — analyzed phishing email header

---
