# PENETRATION TESTING REPORT

### Footprinting & Network Scanning Phases

**W2-PM-FINAL | CYBERSECURITY | NETWORKWALKS**

| Field | Detail |
| --- | --- |
| **Pentester Name (Cybersecurity Professional)** | **Jahmir Vargas** |
| **Program/Batch** | B084-Networkwalks |
| **Date** | 13-14 November 2026 |
| **Modules completed** | W2-PM1 (Multiple Kali Tools)<br>W2-PM4 (theHarvester)<br>W2-PM5 (Zenmap Scanning) |
| **Client/Target** | 1. Networkwalks, networkwalks.com (secured written permission already)<br>2. My own local LAN network (192.168.1.0/24)<br>3. microsoft.com (assigned lab target for W2-PM4, passive public-source OSINT only, no direct contact with the target) |
| **Permission secured from client?** | Yes for networkwalks.com (Letter of Authorization) and for my own LAN. W2-PM4 used only public data sources. |
| **Phases covered** | **Phase 1:** Reconnaissance & Footprinting<br>**Phase 2:** Scanning & Network Discovery<br>**Phase 3-5:** In Progress |

# 1. Liability Disclaimer

I have performed these activities only on the systems & devices where I had secured written permission or the devices/systems that I own myself. The one exception is W2-PM4, where the assigned target (microsoft.com) was queried only through public third-party data sources and was never contacted directly. All these materials are for education and research purpose only. Do not use anything from here to break the law. The instructor, the authors and Networkwalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse can lead to criminal charges, heavy fines, loss of your job and a permanent record. In most countries unauthorised access is a crime even when nothing is damaged.

# 2. Introduction

This report covers footprinting the networkwalks.com domain using multiple Kali Linux tools (W2-PM1), harvesting public email and host information with theHarvester (W2-PM4), and scanning my own local network with Zenmap (W2-PM5). Two modules cover the footprinting phase and one covers the scanning phase, so together they show how an attacker moves from gathering public information to mapping live hosts on a network. It is the Week 2 part of my ongoing internship program at Networkwalks.

All commands were run in Kali Linux, including the Zenmap scan (Zenmap is available on Kali, so no Windows PC was needed). Every step below includes the exact command used, the result I observed, a screenshot as evidence, and a short note on why the finding matters from an attacker's point of view.

# 3. Tools Used

The table below lists each tool used in this report and its purpose.

| Tool | Purpose |
| --- | --- |
| Kali Linux | Operating system used for all reconnaissance and scanning activities |
| WHOIS | Find domain registration details (owner, dates, name servers). |
| whatweb | Fingerprint web technologies (server, CMS, plugins, IP). |
| nslookup | Resolve the domain name to its IP address using DNS. |
| curl -I | Read the HTTP response headers of the website. |
| wafw00f | Detect whether a Web Application Firewall protects the site. |
| dnsrecon | Enumerate all DNS records (NS, MX, SPF, TXT, SRV). |
| theHarvester | Gather emails, sub-domains and hosts from public sources. |
| Zenmap (Nmap GUI) | Scan the local subnet to find live hosts, IPs and MAC addresses. |
| `ip a` / `ip link show` | Local IP address, subnet and MAC address identification (Kali equivalent of Windows `ipconfig`). |

# 4. Activities Performed

## 4.1 Footprinting & Reconnaissance (W2-PM1)

I performed reconnaissance against the `networkwalks.com` domain using six Kali Linux tools: **WHOIS, WhatWeb, Nslookup, Curl, Wafw00f and DNSRecon**. Each tool was used to collect a different type of information about the target. Output was saved to text files with `tee`, and the full outputs are reproduced in Appendix A.

**Task 1: WHOIS** (`whois networkwalks.com`). The registry record shows the domain registered through GoDaddy.com, LLC (IANA ID 146), created on 6 November 2019, last updated on 12 November 2025, and expiring on 6 November 2027. The registry lists the name servers NS6135.HOSTGATOR.COM and NS6136.HOSTGATOR.COM, which reveals that the site is hosted with HostGator. The registrar's own record also lists NS29/NS30.DOMAINCONTROL.COM. The registrant and technical contacts are hidden behind the privacy service Domains By Proxy, LLC, so no personal owner details are exposed. DNSSEC is unsigned, and the domain carries the four client lock statuses (delete, renew, transfer and update prohibited).

**Task 2: WhatWeb** (`whatweb networkwalks.com`). The plain HTTP site returns a 301 redirect to HTTPS, and the HTTPS site (200 OK) runs on Apache. WhatWeb identified **WordPress 7.1.4** and **WordPress Download Manager 3.3.76** from the generator meta tag, jQuery 3.7.1, Google Tag Manager, the page title "Networkwalks Academy", the server IP **192.232.216.135**, and the contact email **info@networkwalks.com**.

**Task 3: Nslookup** (`nslookup networkwalks.com`). Using my local resolver (192.168.1.1), the domain resolved to **192.232.216.135** (a non-authoritative answer), which matches the IP found by WhatWeb.

**Task 4: Curl** (`curl -I https://networkwalks.com`). The server answered `HTTP/1.1 200 OK` with `Server: Apache` and a `Link` header that exposes the WordPress REST API endpoint `/wp-json/`. Other headers disclose `X-Endurance-Cache-Level` and `X-nginx-cache: WordPress`, which hint at the hosting platform's caching layer. The `__wpdm_client` cookie is set with the `secure` and `HttpOnly` flags, and the response carries a `Referrer-Policy` and a `Permissions-Policy` header.

**Task 5: Wafw00f** (`wafw00f networkwalks.com`). Wafw00f v2.3.2 checked `https://networkwalks.com` using 2 requests and reported that the site is **behind ModSecurity (SpiderLabs) WAF**. This is a good sign for the defender, because a WAF filters common web attacks. It also means the WAF product is identifiable from outside, which tells an attacker what kind of filtering rules to expect and plan around.

**Task 6: DNSRecon** (`dnsrecon -d networkwalks.com`). The first attempt failed with `AttributeError: module 'urllib.request' has no attribute 'FancyURLopener'`, a Python 3.14 compatibility bug in the packaged dnsrecon, so I resolved it and re-ran the scan to completion (output in Appendix A). It returned the SOA record (ns6135.hostgator.com, 50.87.144.87), the name servers ns6135.hostgator.com (50.87.144.87) and ns6136.hostgator.com (192.232.216.131), the mail server `mail.networkwalks.com` (192.232.216.135), the A record (192.232.216.135), an SPF TXT record (`v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all`), a Google site-verification TXT record, and eight cPanel autodiscover SRV records. Both name servers reported the DNS software version string **Bind 9.16.23-RH**. The DNSSEC query returned an error ("No answer for DNSSEC query"), consistent with the unsigned DNSSEC status seen in WHOIS.

## 4.2 Footprinting with theHarvester (W2-PM4)

The task was to use theHarvester (version 4.10.1) to find email IDs and sub-domains related to microsoft.com. This is passive reconnaissance: theHarvester queries third-party public sources and does not contact the target.

**Task 1: Baidu source, limit 1000** (`theHarvester -d microsoft.com -l 1000 -b baidu`). The search returned no IPs, no emails, no people and no hosts. The lab guide notes that results vary because data sources change their algorithms over time, so an empty result from Baidu at the time of testing is a valid outcome.

**Task 2: All sources, limit 50** (`theHarvester -d microsoft.com -l 50 -b all`). Many sources failed with "Missing API key" errors (for example bevigil, bitbucket, bufferoverun, builtwith, brave, censys, hunter, shodan and virustotal), so the results reflect only the sources that work without a key. The run still returned:

- **3 email addresses:** dotnet-docker-bot@microsoft.com, opencode@microsoft.com and secure@microsoft.com (role or bot-style accounts rather than named employees).
- **9,965 host names**, many of them wildcard entries (for example `*.api.cfp.microsoft.com`) that appear to come from certificate and DNS data sources.
- **162 IP addresses** and **10 ASNs** (including AS8075, Microsoft's main network).
- **6 interesting URLs**, and no LinkedIn users or people.

The host count is far above the requested limit of 50. The `-l` option appears to limit only the search-engine style sources, while passive DNS and certificate sources returned their full data sets. The complete list is saved in `pm4_task2_all.txt` (summary in Appendix A).

## 4.3 Network Scanning with Zenmap (W2-PM5)

For the scanning activity, I used **Zenmap** on Kali Linux to perform network discovery on my own local network. The task required me to identify my local IP address and subnet, discover live hosts, identify their IP and MAC addresses, and generate a network topology.

I first used `ip a` to identify my local IP address (192.168.1.136) and subnet (192.168.1.0/24). I then entered the subnet into Zenmap, selected the **Ping scan** profile (`nmap -sn 192.168.1.0/24`) and scanned. I also saved the same scan as text with Nmap 7.99 (`nmap -sn 192.168.1.0/24 | tee pm5_pingscan.txt`).

- **Task 4, live hosts:** 7 hosts were live ("256 IP addresses (7 hosts up) scanned in 5.63 seconds"), including my own Kali machine.
- **Task 5, IP addresses:** 192.168.1.1 (router), 192.168.1.50, 192.168.1.141, 192.168.1.152, 192.168.1.205, 192.168.1.212, and 192.168.1.136 (my own Kali machine).
- **Task 6, MAC addresses** (last three bytes masked for privacy): 50:6F:0C:xx:xx:xx (router, Sagemcom), D8:B3:2F:xx:xx:xx (Cloud Network Technology), 80:05:1F:xx:xx:xx (Vizio), 0C:79:55:xx:xx:xx (Hui Zhou Gaoshengda), 94:58:CB:xx:xx:xx (Nintendo) and 8E:07:05:xx:xx:xx (unknown vendor). Nmap does not report a MAC for the scanning host itself, and my own MAC address is withheld from this report for privacy.

The vendor names in the scan show what kinds of devices are on the network (a router, a TV, a games console and personal devices), which is the kind of information an attacker could use to profile a network. Device names that identify individual people were redacted from the screenshots and the appendix, and MAC addresses were masked, to protect the privacy of other users of the network.

After the scan I opened the **Topology** tab, read the legend and saved the topology as a PDF (`pm5_topology.pdf`, included with this submission). I also answered the questions on the Networkwalks lab page.

# 5. Risk Analysis / Impact

Based on the information collected during the footprinting and network scanning activities, I identified the following potential risks.

| **#** | **Risk / Finding** | **Evidence / Observation** | **Potential Impact** | **Risk Level** |
| --- | --- | --- | --- | --- |
| 1 | Web technology information exposed | WhatWeb identified WordPress and WP Download Manager with version numbers | Attackers may use exposed technology/version information to identify software requiring further security review | **● Medium** |
| 2 | Server IP address identifiable | Nslookup resolved the domain to `192.232.216.135` | Provides information about the network location of the web service | **● Low** |
| 3 | HTTP technical information exposed | Curl returned HTTP response headers and exposed `/wp-json/` | May assist technology fingerprinting and further enumeration | **● Low** |
| 4 | WAF technology identifiable | Wafw00f identified the site as protected by ModSecurity (SpiderLabs) WAF | Reveals information about the web application's security architecture | **● Low** |
| 5 | DNS infrastructure information exposed | DNSRecon identified DNS, mail, SPF and cPanel service records and the DNS software version (Bind 9.16.23-RH) | DNS information can help build a broader infrastructure profile | **● Medium** |
| 6 | Large number of hosts, IPs and emails discoverable from public sources | theHarvester (all sources) returned 9,965 hosts, 162 IPs, 10 ASNs and 3 role email addresses for microsoft.com | Public aggregation of host and email data widens an organization's visible attack surface and can feed phishing or targeting | **● Medium** |
| 7 | Multiple live hosts visible on local network | Zenmap identified 7 live hosts on my home LAN, including vendor details for each device, including IoT and entertainment devices (TV, console) | Unknown or unauthorized devices may be present, and IoT devices are often less well maintained | **● Medium** |

**Risk level key:** ● Critical ● Medium ● Low

The risks above are observations from the footprinting and scanning exercises, not confirmed vulnerabilities.

The practical exercises primarily involved information gathering and host discovery. No exploitation or vulnerability validation was performed as part of these modules. Therefore, the presence of information such as a software version, IP address or DNS record does not by itself mean that the system is vulnerable. Further authorized security testing would be required to confirm any actual vulnerability.

# 6. Recommendations

Based on the observations from these activities, I recommend the following security improvements:

1. **Review publicly exposed technology information**
   Organizations should regularly review what information about their web technologies, CMS and plugins is publicly visible.

2. **Keep software updated**
   CMS platforms, plugins and other web technologies should be regularly updated and reviewed against current security advisories.

3. **Review HTTP headers**
   HTTP response headers should be reviewed to determine whether unnecessary technical information is being exposed.

4. **Review DNS records regularly**
   DNS records should be checked periodically to ensure that only required information and services are publicly exposed, and DNS server version banners should be hidden.

5. **Properly configure and monitor the WAF**
   Keep the WAF enabled and tuned, since it blocks naive attacks.

6. **Monitor what public data sources say about the organization**
   Organizations should run tools such as theHarvester against their own domains to see which emails, hosts and sub-domains are publicly discoverable, and reduce unnecessary exposure.

7. **Perform regular internal network discovery**
   Organizations should periodically scan their own networks to identify active devices.

8. **Investigate unknown devices**
   Any unexpected device discovered during network scanning should be investigated and verified.

9. **Maintain network documentation**
   Network topology and device information should be documented and updated regularly.

10. **Perform security testing with authorization**
    Reconnaissance and scanning should only be performed against systems and networks where appropriate authorization has been provided.

# 7. Conclusion

During Week 2 of my Cybersecurity & Ethical Hacking internship, I completed practical activities covering footprinting, reconnaissance and network scanning.

In the footprinting activities, I used six Kali Linux tools to collect information about the networkwalks.com domain, and theHarvester to look for emails and hosts related to microsoft.com. I learned how WHOIS can provide domain information, WhatWeb can identify web technologies, Nslookup can resolve domain names, Curl can inspect HTTP headers, Wafw00f can identify a WAF, DNSRecon can provide additional DNS information, and theHarvester can combine many public sources, although its results depend on which sources are available and which need API keys.

In the network scanning activity, I used Zenmap to identify my local network configuration and discover active hosts. I also collected IP and MAC address information and created a network topology.

I also hit and solved a real tooling problem when dnsrecon failed because of a Python version incompatibility, which showed me that troubleshooting and documenting workarounds is part of the job.

The exercises showed me that information gathering is an important part of cybersecurity. Even before attempting to exploit a system, a security professional can learn a significant amount about an environment by carefully analyzing publicly available information and network responses.

I also learned that technical findings should be documented clearly. A good cybersecurity report should explain what was performed, what was discovered, what the observation means, what risk it may create, and what can be done to reduce that risk.

Finally, I learned that reconnaissance and scanning must always be performed within an authorized scope. These activities were completed as part of the assigned educational cybersecurity lab.

# 8. Evidences Collected

*Screenshots collected as evidence during the activities (stored in the `screenshots/` folder):*

![whois output](screenshots/1_whois.png)

![whatweb output](screenshots/2_whatweb.png)

![nslookup output](screenshots/3_nslookup.png)

![curl output](screenshots/4_curl.png)

![wafw00f output](screenshots/5_wafw00f.png)

![dnsrecon output](screenshots/6_dnsrecon.png)

![theHarvester Baidu output](screenshots/7_theharvester_baidu.png)

![theHarvester all sources output](screenshots/7_theharvester_all.png)

![Zenmap scan and topology (device names redacted)](screenshots/8_zenmap_topology_redacted.png)

Topology PDF: `pm5_topology.pdf` (submitted alongside this report).

The text output of every command is reproduced in Appendix A (section 9).

# 9. Appendix A: Command Outputs

*Text outputs saved during the activities. Long lines are wrapped for printing.*

### W2-PM1 Task 1: whois networkwalks.com

```
$ whois networkwalks.com
Domain Name: NETWORKWALKS.COM
   Registry Domain ID: 2452319255_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.godaddy.com
   Registrar URL: http://www.godaddy.com
   Updated Date: 2025-11-12T10:08:43Z
   Creation Date: 2019-11-06T22:51:46Z
   Registry Expiry Date: 2027-11-06T22:51:46Z
   Registrar: GoDaddy.com, LLC
   Registrar IANA ID: 146
   Registrar Abuse Contact Email: abuse@godaddy.com
   Registrar Abuse Contact Phone: 480-624-2505
   Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
   Domain Status: clientRenewProhibited https://icann.org/epp#clientRenewProhibited
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
   Name Server: NS6135.HOSTGATOR.COM
   Name Server: NS6136.HOSTGATOR.COM
   DNSSEC: unsigned
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of whois database: 2026-11-13T15:15:33Z <<<

[registrar record excerpt]
Domain Name: networkwalks.com
Registry Domain ID: 2452319255_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.godaddy.com
Registrar URL: https://www.godaddy.com
Updated Date: 2025-11-12T05:08:41Z
Creation Date: 2019-11-06T17:51:46Z
Registrar Registration Expiration Date: 2027-11-06T17:51:46Z
Registrar: GoDaddy.com, LLC
Registrar IANA ID: 146
Registrar Abuse Contact Email: abuse@godaddy.com
Registrar Abuse Contact Phone: +1.4806242505
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
Domain Status: clientRenewProhibited https://icann.org/epp#clientRenewProhibited
Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
Registry Registrant ID: Not Available From Registry
Registrant Name: Registration Private
Registrant Organization: Domains By Proxy, LLC
Registrant Street: DomainsByProxy.com
Registrant Street: 100 S. Mill Ave, Suite 1600
Registrant City: Tempe
Registrant State/Province: Arizona
Registrant Postal Code: 85281
Registrant Country: US
Registrant Phone: +1.4806242599
Registrant Phone Ext:
Registrant Fax: 
Registrant Fax Ext:
Registrant Email: https://www.godaddy.com/whois/results.aspx?domain=networkwalks.com&action=contactDomainOwner
Registry Tech ID: Not Available From Registry
Tech Name: Registration Private
Tech Organization: Domains By Proxy, LLC
Tech Street: DomainsByProxy.com
Tech Street: 100 S. Mill Ave, Suite 1600
Tech City: Tempe
Tech State/Province: Arizona
Tech Postal Code: 85281
Tech Country: US
Tech Phone: +1.4806242599
Tech Phone Ext:
Tech Fax: 
Tech Fax Ext:
Tech Email: https://www.godaddy.com/whois/results.aspx?domain=networkwalks.com&action=contactDomainOwner
Name Server: NS6135.HOSTGATOR.COM
Name Server: NS6136.HOSTGATOR.COM
Name Server: NS29.DOMAINCONTROL.COM
Name Server: NS30.DOMAINCONTROL.COM
DNSSEC: unsigned
```

### W2-PM1 Task 2: whatweb networkwalks.com

```
$ whatweb networkwalks.com
http://networkwalks.com [301 Moved Permanently] Apache, Cookies[__wpdm_client], Country[UNITED STATES][US], HTTPServer[Apache], HttpOnly[__wpdm_client], IP[192.232.216.135], RedirectLocation[https://networkwalks.com/], UncommonHeaders[permissions-policy,x-redirect-by,referrer-policy,x-endurance-cache-level,x-nginx-cache]
https://networkwalks.com [200 OK] Apache, Bootstrap[7.1.4], Cookies[__wpdm_client], Country[UNITED STATES][US], Email[info@networkwalks.com], Frame, Google-Tag-Manager, HTML5, HTTPServer[Apache], HttpOnly[__wpdm_client], IP[192.232.216.135], JQuery[3.7.1], MetaGenerator[WordPress 7.1.4,WordPress Download Manager 3.3.76], Open-Graph-Protocol[website], Script[4684NR-IPIB&pidnVar2=50511&prtVar2=7&scvVar2=12,application/json,application/ld+json,module,speculationrules,text/javascript], Title[Networkwalks Academy], UncommonHeaders[permissions-policy,link,referrer-policy,x-endurance-cache-level,x-nginx-cache], WordPress[7.1.4]
https://networkwalks.com/ [200 OK] Apache, Bootstrap[7.1.4], Country[UNITED STATES][US], Email[info@networkwalks.com], Frame, Google-Tag-Manager, HTML5, HTTPServer[Apache], IP[192.232.216.135], JQuery[3.7.1], MetaGenerator[WordPress 7.1.4,WordPress Download Manager 3.3.76], Open-Graph-Protocol[website], Script[4684NR-IPIB&pidnVar2=50511&prtVar2=7&scvVar2=12,application/json,application/ld+json,module,speculationrules,text/javascript], Title[Networkwalks Academy], UncommonHeaders[permissions-policy,link,referrer-policy,x-endurance-cache-level,x-nginx-cache], WordPress[7.1.4]
```

### W2-PM1 Task 3: nslookup networkwalks.com

```
$ nslookup networkwalks.com
Server:		192.168.1.1
Address:	192.168.1.1#53

Non-authoritative answer:
Name:	networkwalks.com
Address: 192.232.216.135
```

### W2-PM1 Task 4: curl -I https://networkwalks.com

```
$ curl -I https://networkwalks.com
HTTP/1.1 200 OK
Date: Fri, 13 Nov 2026 15:16:03 GMT
Server: Apache
Permissions-Policy: private-state-token-redemption=(self "https://www.google.com" "https://www.gstatic.com" "https://recaptcha.net" "https://challenges.cloudflare.com" "https://hcaptcha.com"), private-state-token-issuance=(self "https://www.google.com" "https://www.gstatic.com" "https://recaptcha.net" "https://challenges.cloudflare.com" "https://hcaptcha.com")
Link: <https://networkwalks.com/wp-json/>; rel="https://api.w.org/", <https://networkwalks.com/wp-json/wp/v2/pages/53>; rel="alternate"; title="JSON"; type="application/json", <https://networkwalks.com/>; rel=shortlink
Set-Cookie: __wpdm_client=6eb9ee08c2be03e73efdc6c8f49033e3; path=/; domain=networkwalks.com; secure; HttpOnly
Referrer-Policy: no-referrer-when-downgrade
X-Endurance-Cache-Level: 0
X-nginx-cache: WordPress
Content-Type: text/html; charset=UTF-8
```

### W2-PM1 Task 5: wafw00f networkwalks.com

```
$ wafw00f networkwalks.com | tee pm1_task5_wafw00f.txt

                   ______
                  /      \
                 (  Woof! )
                  \  ____/                      )
                  ,,                           ) (_
             .-. -    _______                 ( |__|
            ()``; |==|_______)                .)|__|
            / ('        /|\                  (  |__|
        (  /  )        / | \                  . |__|
         \(_)_))      /  |  \                   |__|

                    ~ WAFW00F : v2.3.2 ~
    The Web Application Firewall Fingerprinting Toolkit

[*] Checking https://networkwalks.com
[+] The site https://networkwalks.com is behind ModSecurity (SpiderLabs) WAF.
[~] Number of requests: 2
```

### W2-PM1 Task 6: dnsrecon -d networkwalks.com

```
$ dnsrecon -d networkwalks.com
2026-11-14T08:45:50.957627-0700 INFO Starting enumeration for domain: networkwalks.com
2026-11-14T08:45:50.959249-0700 INFO std: Performing General Enumeration against: networkwalks.com...
2026-11-14T08:45:51.243313-0700 ERROR No answer for DNSSEC query for networkwalks.com
2026-11-14T08:45:51.477627-0700 INFO     SOA ns6135.hostgator.com 50.87.144.87
2026-11-14T08:45:51.630123-0700 INFO     NS ns6136.hostgator.com 192.232.216.131
2026-11-14T08:45:51.741742-0700 INFO     Bind Version for 192.232.216.131 "9.16.23-RH"
2026-11-14T08:45:51.741971-0700 INFO     NS ns6135.hostgator.com 50.87.144.87
2026-11-14T08:45:51.845415-0700 INFO     Bind Version for 50.87.144.87 "9.16.23-RH"
2026-11-14T08:45:52.638435-0700 INFO     MX mail.networkwalks.com 192.232.216.135
2026-11-14T08:45:53.457411-0700 INFO     A networkwalks.com 192.232.216.135
2026-11-14T08:45:54.007981-0700 INFO     TXT networkwalks.com google-site-verification=rr04eRmqHoWY3XemnizDNVK4q75X-Ij-mjgEeg-UsYI
2026-11-14T08:45:54.008239-0700 INFO     TXT networkwalks.com v=spf1 +a +mx +ip4:50.87.144.87 +include:websitewelcome.com ~all
2026-11-14T08:45:54.132383-0700 INFO Enumerating SRV Records
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.204.15 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.203.9 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.204.11 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.203.11 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.203.14 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.203.15 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.204.14 443
    SRV _autodiscover._tcp.networkwalks.com cpanelemaildiscovery.cpanel.net 184.94.204.9 443
INFO 8 Records Found
INFO Completed enumeration for domain: networkwalks.com
```

### W2-PM4 Task 1: theHarvester with Baidu

```
$ theHarvester -d microsoft.com -l 1000 -b baidu
[*] Target: microsoft.com 

[*] Searching Baidu. 

[*] No IPs found.

[*] No emails found.

[*] No people found.

[*] No hosts found.
```

### W2-PM4 Task 2: theHarvester with all sources

```
$ theHarvester -d microsoft.com -l 50 -b all   (summary; full output in pm4_task2_all.txt)
[*] Target: microsoft.com
[!] Missing API key for: bevigil, Bitbucket, bufferoverun, BuiltWith, Brave, Censys, Hunter, Shodan,
    VirusTotal and others (these sources were skipped)

[*] ASNS found: 10
--------------------
AS13335
AS133618
AS16625
AS206834
AS209242
AS20940
AS40034
AS8068
AS8070
AS8075

[*] Interesting Urls found: 6 (long OAuth login URLs omitted here)
[*] IPs found: 162 (list in pm4_task2_all.txt)

[*] Emails found: 3
----------------------
dotnet-docker-bot@microsoft.com
opencode@microsoft.com
secure@microsoft.com

[*] Hosts found: 9965
---------------------
*.api.cfp.microsoft.com
*.bigcatalog-ppe.commerce.microsoft.com
*.bizextsvc-int.microsoft.com
*.canary.mfavalidation.auth.microsoft.com
*.canary.testmymfa.auth.microsoft.com
*.ckms.prod.communication.microsoft.com
*.cloudcare-aks-ppe.caresupplychain.microsoft.com
*.cloudcare-aks-prd.caresupplychain.microsoft.com
*.console-m365d.microsoft.com
*.copilotstudio.microsoft.com
... (9965 host names in total)
```

### W2-PM5: Zenmap / Nmap ping scan

```
$ nmap -sn 192.168.1.0/24   (names redacted, MACs masked)
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-13 21:59 -0700
Nmap scan report for osync.lan (192.168.1.1)
Host is up (0.0060s latency).
MAC Address: 50:6F:0C:xx:xx:xx (Sagemcom Broadband SAS)
Nmap scan report for [redacted-device] (192.168.1.50)
Host is up (0.20s latency).
MAC Address: D8:B3:2F:xx:xx:xx (Cloud Network Technology Singapore PTE.)
Nmap scan report for 192.168.1.141
Host is up (0.24s latency).
MAC Address: 80:05:1F:xx:xx:xx (Vizio)
Nmap scan report for [redacted-device] (192.168.1.152)
Host is up (0.096s latency).
MAC Address: 0C:79:55:xx:xx:xx (Hui Zhou Gaoshengda Technology)
Nmap scan report for 192.168.1.205
Host is up (0.20s latency).
MAC Address: 94:58:CB:xx:xx:xx (Nintendo)
Nmap scan report for [redacted-device] (192.168.1.212)
Host is up (0.18s latency).
MAC Address: 8E:07:05:xx:xx:xx (Unknown)
Nmap scan report for kali.lan (192.168.1.136)
Host is up.
Nmap done: 256 IP addresses (7 hosts up) scanned in 5.63 seconds
```

-End-
