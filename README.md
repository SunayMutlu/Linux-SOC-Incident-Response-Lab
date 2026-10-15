# 🛡️ Enterprise SOC Incident Investigation & Response Lab

![Status](https://img.shields.io/badge/Status-Resolved-success)
![Platform](https://img.shields.io/badge/Platform-Ubuntu%20Server-orange)
![Attacker](https://img.shields.io/badge/Attacker-Kali%20Linux-red)
![Framework](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-blue)

## 📌 Executive Summary
This project demonstrates an end-to-end security incident lifecycle conducted within an isolated virtualized enterprise network. As a SOC Analyst / Incident Responder, I simulated, monitored, triaged, and contained two distinct cyber attacks against a Linux-based server:
1. Credential Access via SSH Brute-Force (MITRE ATT&CK T1110.001)
2. Web Application Exploitation & Discovery via SQL Injection & Path Traversal (MITRE ATT&CK T1190 & T1083)

Both incidents were investigated using live kernel and application telemetry (auditd, systemd-journald, OpenSSH, and Apache2), followed by immediate tactical containment using Linux Kernel Packet Filtering (iptables).

---

## 🏗️ Environment Architecture
* Virtualization: Oracle VirtualBox (Isolated Host-Only Network: 192.168.56.0/24)
* Blue Team (Target Server): Ubuntu Server 26.04 LTS (192.168.56.105)
  - Services: OpenSSH (Port 22), Apache2 HTTP Server (Port 80)
  - Telemetry Sensors: auditd, systemd-journald, iptables
* Red Team (Attacker): Kali Linux (192.168.56.112)
  - Tools: Nmap, Hydra, cURL
* Management Station: Windows 11 Host (192.168.56.106)

---

## 🔍 Incident 1: SSH Brute-Force Attack (MITRE T1110.001)

### 1. Reconnaissance & Initial Probe
The threat actor conducted port reconnaissance using Nmap to discover open administrative interfaces:
Command: nmap -p 22 192.168.56.105
Result: 22/tcp open ssh

### 2. Automated Password Spraying & Brute Force
The attacker deployed Hydra against the vboxuser account utilizing automated wordlists across 4 parallel threads.

### 3. Detection & Live Log Triage
Monitoring OpenSSH daemon logs in real-time (sudo journalctl -u ssh -f) captured multiple rapid authentication failures:

Evidence Logs:
- Invalid user sahte_hacker from 192.168.56.112 port 53724
- pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.56.112 user=vboxuser
- Failed password for vboxuser from 192.168.56.112 port 33438 ssh2
- Failed password for vboxuser from 192.168.56.112 port 33442 ssh2
- srclimit_penalise: 192.168.56.112/32: activating ipv4 penalty of 18 seconds for penalty: exceeded LoginGraceTime

Key Forensic Findings:
* Source IP: 192.168.56.112
* Target Account: vboxuser and invalid alias sahte_hacker
* Indicator of Compromise (IoC): 4 connection attempts within 3 seconds, triggering OpenSSH LoginGraceTime timeouts and automatic rate penalties.

### 4. Active Containment (Incident Response)
To mitigate ongoing socket exhaustion and credential compromise, a host-based firewall rule was applied directly to the Linux Kernel to drop all inbound traffic from the malicious IP:
Command: sudo iptables -I INPUT -s 192.168.56.112 -j DROP

### 5. Post-Mitigation Verification
A connectivity audit was conducted from the attacker's machine:
Command: ping -c 3 192.168.56.105
Result: 3 packets transmitted, 0 received, 100% packet loss, time 2057ms
Outcome: The attacker was successfully blackholed while administrative access from the management station (192.168.56.106) remained fully operational.

---

## 🌐 Incident 2: Web Application Exploitation (MITRE T1190 & T1083)

### 1. Attack Vectors (SQL Injection & Path Traversal)
The threat actor issued malicious HTTP GET requests targeting Port 80:
1. SQL Injection Probe: Testing boolean logic bypass (?id=1' OR 1=1--)
2. Sensitive File Disclosure: Attempting path traversal to access /etc/passwd

### 2. Web Access Log Analysis (/var/log/apache2/access.log)
Evidence Logs:
- 192.168.56.112 - - [15/Oct/2026:13:19:37 +0300] "GET /?id=1'%20OR%201=1-- HTTP/1.1" 200 10927 "-" "Saldirgan_Boooooot"
- 192.168.56.112 - - [15/Oct/2026:13:20:03 +0300] "GET /etc/passwd HTTP/1.1" 404 477 "-" "Saldirgan_Bot"

Key Forensic Findings:
* User-Agent Anomaly: Non-standard agent headers (Saldirgan_Bot, Saldirgan_Boooooot) indicating scripted automation.
* SQLi Pattern: Query string containing ?id=1'%20OR%201=1-- attempting SQL logic manipulation.
* Directory Traversal: Server returned HTTP 404 Not Found, confirming the web root restricted unauthorized file system exposure.

---

## 🛡️ Strategic Remediation & Hardening Recommendations

1. SSH Hardening:
   - Disable password-based authentication; enforce asymmetric Ed25519 public-key authentication.
   - Relocate default SSH port from 22 to a non-standard high port.
2. Automated Threat Banning:
   - Implement Fail2ban to automatically trigger dynamic iptables/nftables block rules upon consecutive authentication failures.
3. Web Application Firewall (WAF):
   - Deploy ModSecurity with OWASP Core Rule Set (CRS) on Apache to inspect and drop SQLi/LFI payloads before hitting application logic.
4. Kernel-Level Audit Rules:
   - Maintain persistent /etc/audit/rules.d/ configurations monitoring modifications to /etc/passwd and /etc/shadow.

---
Incident Status: RESOLVED  
Report Prepared By: Security Operations Engineer / SOC Analyst
