# Footprinting & Network Scanning: Week 2 Penetration Testing Report

**Cybersecurity-Footprinting-and-Scanning-Jahmir-Vargas-Week2**

Reconnaissance, OSINT and network discovery practice using Kali Linux tools, completed as Week 2 of the Networkwalks Cybersecurity program.

![Skill](https://img.shields.io/badge/Skill-Cybersecurity-red) ![Kali](https://img.shields.io/badge/Kali%20Linux-2026-black) ![Phase](https://img.shields.io/badge/Phases-Recon%20%26%20Scanning-teal) ![Nmap](https://img.shields.io/badge/Nmap-7.99-blue) ![Program](https://img.shields.io/badge/Networkwalks-B084-orange)

---

## Project Overview

This project covers the first two phases of a penetration test:

1. **Reconnaissance & Footprinting:** collecting public information about a target without attacking it.
2. **Scanning & Network Discovery:** mapping live hosts on a network.

Together they show how an attacker moves from gathering public information to mapping a network, and what defenders can do to reduce that exposure.

**Author:** Jahmir Vargas
**Date:** 13-14 November 2026

The full write-up is in [`REPORT.md`](REPORT.md) (also available as a PDF in this repo).

---

## Authorization and Scope

| Target | Authorization |
|---|---|
| Networkwalks domain (W2-PM1) | Written permission secured (Letter of Authorization) |
| My own home LAN (W2-PM5) | Devices and network I own |
| Assigned lab domain (W2-PM4) | Passive public-source OSINT only. The target was never contacted directly. |

No exploitation or vulnerability validation was performed. Findings are observations, not confirmed vulnerabilities.

---

## Modules Completed

| Module | Topic | Tools |
|---|---|---|
| W2-PM1 | Footprinting with multiple Kali tools | WHOIS, WhatWeb, nslookup, curl, wafw00f, dnsrecon |
| W2-PM4 | Email and host discovery from public sources | theHarvester 4.10.1 |
| W2-PM5 | Local network discovery | Zenmap / Nmap 7.99, `ip a` |

---

## Key Findings

### W2-PM1: Footprinting an authorized domain

- **WHOIS:** registered through GoDaddy, created in 2019, expiring in 2027. Name servers point to HostGator. Owner details are hidden behind a privacy service, and DNSSEC is unsigned.
- **WhatWeb:** Apache web server running WordPress with the WordPress Download Manager plugin, with version numbers visible in the page.
- **nslookup:** resolved the domain to its hosting IP, matching the WhatWeb result.
- **curl -I:** response headers expose the WordPress REST API path and hints about the hosting platform's caching layer.
- **dnsrecon:** enumerated SOA, NS, MX, SPF, TXT and cPanel autodiscover SRV records, and the DNS software version banner (BIND).

### W2-PM4: theHarvester (passive OSINT)

- The Baidu-only search returned nothing, which is a valid result because source results change over time.
- The all-sources search returned **3 role-based email addresses, 9,965 host names, 162 IP addresses and 10 ASNs**, even though many sources were skipped for missing API keys.
- The `-l` limit applied only to search-engine style sources. Passive DNS and certificate sources returned their full data sets.

### W2-PM5: Zenmap ping scan of my own LAN

- Command: `nmap -sn 192.168.1.0/24`
- **7 live hosts** found in about 5.6 seconds, including my own Kali machine.
- MAC vendor lookups revealed the device types (router, TV, games console, personal devices). Device names were redacted and MAC addresses masked in this repo to protect other network users.
- The Zenmap Topology view was exported as a PDF.

---

## Risk Summary

| # | Finding | Level |
|---|---|---|
| 1 | Web technology and version information exposed | Medium |
| 2 | Server IP address identifiable | Low |
| 3 | HTTP technical information exposed | Low |
| 4 | WAF technology identifiable | Low |
| 5 | DNS infrastructure and version banner exposed | Medium |
| 6 | Large number of hosts, IPs and emails discoverable from public sources | Medium |
| 7 | Multiple live hosts, including IoT devices, visible on the local network | Medium |

---

## Troubleshooting

**dnsrecon crashed on start**

```
AttributeError: module 'urllib.request' has no attribute 'FancyURLopener'
```

This is a Python 3.14 compatibility bug in the packaged dnsrecon, which relies on a class removed from newer Python versions. I resolved it and re-ran the scan to completion. *(Add the exact fix you used here.)*

**theHarvester "Missing API key" errors**

Many sources need an API key and were skipped. The results therefore reflect only keyless sources, which I noted in the report.

---

## Repository Structure

```
.
├── README.md
├── REPORT.md                 # full penetration testing report
├── Week2_Pentest_Report_Jahmir_Vargas.pdf
├── screenshots/              # evidence for each task
└── outputs/
    ├── pm4_task2_all.txt     # full theHarvester output
    ├── pm5_pingscan.txt      # Nmap ping scan (redacted)
    └── pm5_topology.pdf      # Zenmap topology export
```

---

## Commands Used

```bash
whois <domain>
whatweb <domain>
nslookup <domain>
curl -I https://<domain>
wafw00f <domain>
dnsrecon -d <domain>

theHarvester -d <domain> -l 1000 -b baidu
theHarvester -d <domain> -l 50 -b all

ip a
nmap -sn 192.168.1.0/24
```

---

## What I Learned

- Each recon tool reveals a different layer: ownership (WHOIS), technology (WhatWeb), network location (nslookup), server behavior (curl), protections (wafw00f) and infrastructure (dnsrecon).
- Passive OSINT can expose a large attack surface without touching the target.
- Ping scans are quick and reveal device types through MAC vendor data.
- Tools break, and documenting a workaround is part of the job.
- Reconnaissance and scanning must stay inside an authorized scope.

---

## Legal and Ethical Notice

This work is for education and research only. Hacking is only legal when I test systems I own, have written permission to test, or am working under a signed agreement with an agreed scope. Everything outside those cases is illegal.
