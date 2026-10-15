# 🛡️ Enterprise SOC Incident Investigation & Response Lab

![Status](https://img.shields.io/badge/Status-Resolved-success)
![Platform](https://img.shields.io/badge/Platform-Ubuntu%20Server-orange)
![Attacker](https://img.shields.io/badge/Attacker-Kali%20Linux-red)
![Framework](https://img.shields.io/badge/Framework-MITRE%20ATT%26CK-blue)

## 📌 Executive Summary
This project demonstrates an end-to-end security incident lifecycle conducted within an isolated virtualized enterprise network. As a SOC Analyst / Incident Responder, I simulated, monitored, triaged, and contained two distinct cyber attacks against a Linux-based server:
1. **Credential Access via SSH Brute-Force (T1110.001)**
2. **Web Application Exploitation & Discovery via SQL Injection & Path Traversal (T1190 & T1083)**

Both incidents were investigated using live kernel and application telemetry (`auditd`, `systemd-journald`, `OpenSSH`, and `Apache2`), followed by immediate tactical containment using Linux Kernel Packet Filtering (`iptables`).

---

## 🏗️ Environment Architecture
* **Virtualization:** Oracle VirtualBox (Isolated Host-Only Network: `192.168.56.0/24`)
* **Blue Team (Target Server):** Ubuntu Server 26.04 LTS (`192.168.56.105`)
  * Services: OpenSSH (Port 22), Apache2 HTTP Server (Port 80)
  * Telemetry Sensors: `auditd`, `systemd-journald`, `iptables`
* **Red Team (Attacker):** Kali Linux (`192.168.56.112`)
  * Tools: Nmap, Hydra, cURL
* **Management Station:** Windows 11 Host (`192.168.56.106`)

---

## 🔍 Incident 1: SSH Brute-Force Attack (MITRE T1110.001)

### 1. Reconnaissance & Initial Probe
The threat actor conducted port reconnaissance using `nmap` to discover open administrative interfaces:
```bash
# Red Team Recon
nmap -p 22 192.168.56.105
# Result: 22/tcp open ssh
