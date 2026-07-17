# Day 10 — Reconnaissance: Passive Recon, OSINT & Shodan
**Date:** June 22, 2026  
**Goal:** Understand how attackers gather intelligence and how defenders use same techniques

---

## 1. What is Reconnaissance?

Reconnaissance is the first phase of any attack gathering information
about a target before launching an attack. Understanding recon helps
SOC analysts know what attackers are looking for and what to protect.

### Reconnaissance in the Attack Lifecycle
Reconnaissance → Weaponization → Delivery → Exploitation
→ Installation → C2 → Actions on Objectives

(Lockheed Martin Cyber Kill Chain — recon is always Step 1)

---

## 2. Passive vs Active Reconnaissance

| Feature          | Passive Reconnaissance              | Active Reconnaissance               |
|------------------|-------------------------------------|-------------------------------------|
| Definition       | Gathering info without touching target | Directly interacting with target |
| Target awareness | Target has NO idea                  | Target may detect it                |
| Legality         | Always legal                        | May be illegal without permission   |
| Traces left      | None on target systems              | Logs on target systems              |
| Examples         | WHOIS, Google, Shodan, DNS lookup   | Nmap scan, ping, banner grabbing    |
| Tools            | Shodan, DNSDumpster, WHOIS          | Nmap, Netcat, Nikto                 |

---

## 3. OSINT — Open Source Intelligence

OSINT is gathering information from publicly available sources.
Both attackers (for targeting) and defenders (for threat intel) use OSINT.

### OSINT Sources
| Source Type      | Examples                                        |
|------------------|-------------------------------------------------|
| DNS Records      | WHOIS, nslookup, DNSDumpster                   |
| Search Engines   | Google dorking, Bing                            |
| Social Media     | LinkedIn (employee names, tech stack), Twitter  |
| Job Postings     | Reveal tech stack, software versions used       |
| Shodan           | Internet-connected devices, exposed services    |
| Certificate logs | crt.sh — find subdomains via SSL certificates  |
| Web archives     | archive.org — see old versions of websites     |
| GitHub           | Leaked credentials, API keys, config files      |

---

## 4. DNS Enumeration

### Key DNS Record Types for Recon
| Record | Command                          | What it reveals                    |
|--------|----------------------------------|------------------------------------|
| A      | `nslookup domain.com`            | IPv4 address of domain             |
| MX     | `nslookup -type=MX domain.com`   | Mail servers — useful for phishing |
| NS     | `nslookup -type=NS domain.com`   | Name servers                       |
| TXT    | `nslookup -type=TXT domain.com`  | SPF, DKIM, verification records    |
| CNAME  | `nslookup -type=CNAME sub.domain`| Alias records, cloud services used |

### Commands Practiced
```bash
# Basic DNS lookup
nslookup google.com

# Find mail servers
nslookup -type=MX google.com

# Find name servers
nslookup -type=NS google.com

# Find TXT records (SPF, DKIM)
nslookup -type=TXT google.com

# WHOIS lookup
whois google.com
```

### WHOIS — What it reveals
- Domain registrar and registration date
- Expiry date (attackers target expiring domains)
- Name servers
- Registrant contact (often hidden by privacy protection)
- Country of registration

---

## 5. Key OSINT Tools

### Shodan (shodan.io)
The "search engine for hackers" — indexes internet-connected devices.

**What Shodan reveals:**
- Open ports and running services on any IP
- Software versions (often outdated/vulnerable)
- Geographic location of servers
- SSL certificate details
- Default credentials warnings

**Useful Shodan searches:**
```bash
port:3389 country:NP          → RDP exposed in Nepal
port:22 country:NP            → SSH exposed in Nepal
apache country:NP             → Apache servers in Nepal
"default password" country:NP → Devices with default passwords
port:445 country:NP           → SMB exposed in Nepal (EternalBlue risk)
org:"Nepal Telecom"           → NTC infrastructure
```
> SOC tip: Run Shodan searches on your own organization's IP range
> to see what you're exposing to the internet. If you can see it,
> attackers can too.

### DNSDumpster (dnsdumpster.com)
- Visual DNS map of any domain
- Shows all subdomains, IPs, MX records
- Free, no login required
- Great for understanding attack surface

### crt.sh (crt.sh)
- Certificate Transparency logs
- Find subdomains via SSL certificates
- Search: `%.targetdomain.com`
- Reveals hidden subdomains often missed by other tools

### URLScan.io (urlscan.io)
- Scans and analyzes websites
- Shows all requests a page makes
- Detects malicious content
- Great for analyzing suspicious URLs safely

### Google Dorking
Special Google search operators to find sensitive info:

| Dork                          | Finds                                    |
|-------------------------------|------------------------------------------|
| `site:target.com`             | All indexed pages of a domain            |
| `site:target.com filetype:pdf`| PDF files on domain                      |
| `site:target.com inurl:admin` | Admin pages                              |
| `intitle:"index of"`          | Open directory listings                  |
| `"@target.com" site:linkedin` | Employee emails on LinkedIn              |
| `site:github.com "target.com" "password"` | Leaked credentials on GitHub |

---

## 6. Subdomain Enumeration

Finding subdomains expands attack surface knowledge:

```bash
# Using nslookup manually
nslookup mail.target.com
nslookup vpn.target.com
nslookup admin.target.com

# Online tools
# dnsdumpster.com — visual map
# crt.sh — SSL certificate search
# subdomainfinder.c99.nl — free online tool
```

Common subdomains worth checking:
- mail.domain.com, vpn.domain.com, admin.domain.com
- dev.domain.com, staging.domain.com (often less secure)
- api.domain.com, portal.domain.com

---
---

## 7. Recon from a Defender's Perspective

As a SOC analyst, use recon techniques to:

| Task                          | Tool                    | Why                                      |
|-------------------------------|-------------------------|------------------------------------------|
| Find exposed services         | Shodan                  | See what attackers see about your org    |
| Map attack surface            | DNSDumpster             | Know all subdomains before attacker does |
| Check for leaked credentials  | GitHub search           | Find exposed API keys, passwords         |
| Verify email security         | nslookup TXT records    | Check SPF/DKIM/DMARC configured          |
| Monitor domain expiry         | WHOIS                   | Prevent domain hijacking                 |

---

## 8. Key Takeaways for SOC Work
- Recon = Step 1 of every attack — understanding it = better defense
- Shodan shows what attackers see — run it on your own org's IPs
- Google dorking finds sensitive files exposed accidentally
- Subdomains are often less secured than main domain — check them
- GitHub leaks are real — search your org name + "password" on GitHub
- crt.sh finds subdomains that don't appear in DNS records

---

## TryHackMe
- [x] Passive Reconnaissance room — completed

## Tools Used Today
- WHOIS — domain registration lookup
- nslookup — DNS record enumeration
- Shodan — internet-exposed device search
- DNSDumpster — visual DNS mapping
- URLScan.io — website analysis
- crt.sh — subdomain via SSL certificates

---
