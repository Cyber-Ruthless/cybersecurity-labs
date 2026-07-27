# Metasploit Framework — Port Scanner & Subnet Enumeration Lab

Date: 2026-07-08
Attacker Machine: Parrot OS
Target Subnet: 10.10.1.5-23
Tool: Metasploit Framework v6.4.71
Environment: VMware isolated lab network

---

## Objective
Use Metasploit Framework's built-in TCP port scanner
to enumerate live hosts across an internal subnet and
chain reconnaissance results into the exploitation
pipeline.

---

## Why Metasploit for Scanning?

Unlike standalone Nmap, scanning inside Metasploit
allows discovered hosts to be passed directly into
exploit modules without leaving the framework.
This is the recon-to-exploitation pipeline used in
real penetration testing engagements.

---

## Step 1 — Launch Metasploit

msfconsole

---

## Step 2 — Search for Port Scanner Modules

search portscan

Modules found:
- auxiliary/scanner/portscan/xmas
- auxiliary/scanner/portscan/ack
- auxiliary/scanner/portscan/tcp
- auxiliary/scanner/portscan/syn

Selected: auxiliary/scanner/portscan/tcp (module 5)

---

## Step 3 — Configure the Module

use 5

set PORTS 80
set RHOSTS 10.10.1.5-23
set THREADS 50
run

---

## Step 4 — Results

10.10.1.19:80 — TCP OPEN
10.10.1.22:80 — TCP OPEN
Scanned 19 of 19 hosts (100% complete)
Auxiliary module execution completed

---

## Hosts Confirmed

| IP | Port | Status | Role |
|----|------|--------|------|
| 10.10.1.19 | 80 | OPEN | Windows Server 2019 Web Server |
| 10.10.1.22 | 80 | OPEN | Windows Server 2022 Domain Controller |

---

## Integration With Nmap Results

Both hosts were previously identified during Nmap
reconnaissance. Confirming them inside Metasploit
means they are now ready to be passed directly
into exploit modules.

Nmap identified:
- 10.10.1.19 — Anonymous FTP enabled, IIS 10.0
- 10.10.1.22 — Active Directory DC, RDP exposed

Next step — exploitation.

---

## Lessons Learned

1. Metasploit's scanner modules are faster than
   Nmap for quick port confirmation across subnets
2. THREADS 50 dramatically speeds up scanning
3. Chaining recon tools into Metasploit creates
   a seamless recon-to-exploitation workflow
4. Always confirm Nmap findings with a second tool

---

## Tools Used
- Metasploit Framework v6.4.71
- Parrot Security OS

---

## Next Steps
- Exploit anonymous FTP on 10.10.1.19
- Enumerate Active Directory on 10.10.1.22
- Run vulnerability scripts against open services


---

## Screenshots

### Portscan Modules


![Portscan Modules](../images/photo_2026-07-27_03-39-44.jpg)



### Scan Results


![Scan Results](../images/photo_2026-07-27_03-39-58.jpg)



### Scan Complete


![Scan Complete](../images/photo_2026-07-27_03-40-02.jpg)


