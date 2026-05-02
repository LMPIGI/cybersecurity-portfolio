# Project 01 — Nmap Reconnaissance & Forensic Artefact Analysis

**Date:** 2026-05-02  
**Tools:** Nmap 7.98, Windows Event Viewer, auditpol  
**Environment:** Kali Linux (attacker) → Windows 10 (victim), isolated host-only network

---

## Objective
Simulate a basic network reconnaissance attack from Kali Linux against a Windows 10 target, then identify and analyse the forensic artefacts left behind in Windows Security logs.

---

## Methodology

### 1. Reconnaissance
Ran an Nmap default scan from Kali to discover open ports on the Windows target:nmap 192.168.44.128

**Findings:** 3 open ports discovered
- 135/tcp — msrpc (Microsoft RPC)
- 139/tcp — netbios-ssn
- 445/tcp — microsoft-ds (SMB)

### 2. Service version detection
nmap -sV 192.168.44.128
**Findings:** Confirmed Windows OS. SMB port 445 open and accessible.

---

## Forensic Analysis

### Enabling audit logging
Windows Filtering Platform connection logging was not enabled by default. Enabled it via: auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable
### Evidence found — Event ID 5156
After re-running the scan, Windows Security logs recorded every connection attempt with the following details:

| Field | Value |
|-------|-------|
| Event ID | 5156 |
| Source Address | 192.168.44.129 (Kali) |
| Destination Port | 445 (SMB) |
| Direction | Inbound |
| Timestamp | 2026-05-02 09:54:57 AM |
| Computer | DESKTOP-96F5GKI |

---

## Conclusion
A single Nmap scan from an attacker machine generated over 2,000 Event ID 5156 entries in the Windows Security log. Each entry recorded the attacker's IP address, the targeted port, and an exact timestamp — sufficient evidence to identify the source and timing of a reconnaissance attack.

**Key forensic takeaway:** Port 445 (SMB) being open and logged as a target would be an immediate red flag in a real investigation, as it is the entry point for exploits like EternalBlue and WannaCry ransomware.

---

## Next Steps
- Attempt SMB exploitation using Metasploit
- Capture and analyse network traffic with Wireshark during the attack
- Document additional artefacts left in Windows registry and prefetch files
