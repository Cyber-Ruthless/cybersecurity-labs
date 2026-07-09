# Nmap Scanning Techniques Lab

Date: 2026-07-08
Attacker Machine: Parrot OS (10.10.1.x)
Targets:
- 10.10.1.19 — www.goodshopping.com (Windows Server 2019)
- 10.10.1.22 — SERVER2022.CEH.com (Windows Server 2022 DC)
- 10.10.1.13 — Linux DNS Server
- 10.10.1.11 — Unknown (heavily filtered)
Tool: Nmap 7.94SVN
Environment: VMware isolated lab network

---

## Objective
Demonstrate multiple Nmap scanning techniques used during
penetration testing reconnaissance to discover hosts,
enumerate services, identify operating systems, and
evade detection mechanisms.

---

## Targets Discovered

| IP | Hostname | OS | Role |
|----|----------|----|------|
| 10.10.1.19 | www.goodshopping.com | Windows Server 2019 | Web/FTP Server |
| 10.10.1.22 | SERVER2022.CEH.com | Windows Server 2022 | Active Directory DC |
| 10.10.1.13 | N/A | Linux 2.6.32 | DNS Server |
| 10.10.1.11 | N/A | Unknown | Heavily Firewalled |

---

## Scan 1 — TCP ACK Ping Scan (Host Discovery)

### Purpose
Discover live hosts on the network using ACK packets
on common ports to bypass simple firewalls.

### Command

### Why It Matters
ACK ping works where ICMP is blocked. Useful for
discovering hosts behind basic firewall rules.

---

## Scan 2 — ARP Ping Scan

### Purpose
Fast host discovery on local network using ARP requests.

### Command

### Why It Matters
ARP ping is the most reliable discovery method on
local subnets — cannot be blocked at layer 3.

---

## Scan 3 — Service Version Detection

### Purpose
Identify exact versions of running services to find
known vulnerabilities.

### Command

### Key Results

| Port | Service | Version |
|------|---------|---------|
| 53/tcp | domain | Simple DNS Plus |
| 80/tcp | http | Microsoft IIS 10.0 |
| 88/tcp | kerberos-sec | Microsoft Windows Kerberos |
| 389/tcp | ldap | Microsoft AD LDAP (CEH.com) |
| 445/tcp | microsoft-ds | Windows Server 2022 Standard |
| 3389/tcp | ms-wbt-server | Microsoft Terminal Services |

### Why It Matters
Version detection reveals exact software versions,
enabling targeted exploitation using known CVEs.

---

## Scan 4 — OS Detection

### Purpose
Fingerprint the target operating system using TCP/IP
stack analysis.

### Command

nmap -A 10.10.1.22
### Key Findings on 10.10.1.22

OS: Windows Server 2022 Standard 20348
Domain: CEH.com
Computer Name: Server2022
SMB signing: enabled and required
RDP Certificate: Server2022.CEH.com

### Why It Matters
OS detection guides exploit selection — different
vulnerabilities exist for different OS versions.

---

## Scan 5 — Aggressive Scan

### Purpose
Combine OS detection, version scanning, script scanning,
and traceroute in one command.

### Command

### Why It Matters
Aggressive scan gives the most complete picture
of a target in a single command. Noisy but thorough.

---

## Scan 6 — MAC Address Spoofing

### Purpose
Disguise attacker identity by randomizing MAC address
during scanning to evade network monitoring.

### Command

nmap -sS -Pn --spoof-mac 0 10.10.1.22

### Why It Matters
MAC spoofing makes it harder for network monitoring
tools and IDS to attribute scans to the real attacker.

---

## Scan 7 — Bad Checksum Scan

### Purpose
Send packets with invalid TCP/UDP checksums to test
if the target or firewall validates packet integrity.

### Command

nmap --badsum 10.10.1.22

### Why It Matters
If a host responds to bad checksum packets, it means
the OS is not properly validating packets — a sign
of a weak network stack or misconfigured firewall.

---

## Scan 8 — Source Port Manipulation

### Purpose
Disguise scan traffic as coming from a trusted port
(port 80) to bypass firewall rules that allow HTTP.

### Command

nmap -g 80 10.10.1.19

### Critical Finding
Anonymous FTP login allowed on port 21!
This is a serious misconfiguration.

### Why It Matters
Firewalls often allow traffic from port 80.
Using port 80 as source port can bypass ACL rules.

---

## Scan 9 — MTU Fragmentation Scan

### Purpose
Fragment packets into smaller chunks to evade
signature-based IDS/IPS detection systems.

### Command

nmap --mtu 8 10.10.1.22

### Why It Matters
Packet fragmentation can bypass IDS systems that
only inspect complete packets. MTU of 8 breaks
packets into very small fragments.

---

## Scan 10 — Packet Fragmentation

### Purpose
Split scan packets into tiny fragments to evade
deep packet inspection.

### Command

nmap -f 10.10.1.22

### Why It Matters
Similar to MTU scan but uses Nmap's built-in
fragmentation. Effective against older IDS systems.

---

## Scan 11 — Decoy Scan

### Purpose
Generate fake source IPs to hide real attacker IP
among multiple decoy addresses.

### Command

nmap -D RND:10 10.10.1.22

### Why It Matters
Security teams monitoring logs will see 11 different
source IPs making it extremely difficult to identify
the real attacker.

---

## Scan 12 — SMB OS Discovery Script

### Purpose
Use Nmap NSE script to extract detailed Windows
information via SMB protocol.

### Command

nmap --script smb-os-discovery.nse 10.10.1.19

### Why It Matters
SMB enumeration reveals domain information, computer
names, and Windows build numbers without authentication.

---

## Scan 13 — Subnet Wide Aggressive Scan

### Purpose
Discover and enumerate all live hosts across the
entire subnet simultaneously.

### Command

nmap -Pn -sS -A -oX Test 10.10.1.0/24
