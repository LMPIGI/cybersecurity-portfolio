# Project 04 - Prefetch & Registry Forensics

**Date:** 2026-06-06  
**Tools:** Windows Prefetch, PowerShell, Registry Editor  
**Environment:** Windows 10 (victim) post-intrusion analysis

---

## Objective
Investigate the forensic artefacts left behind on the Windows filesystem after a Meterpreter backdoor was executed, using prefetch files, registry run keys, and filesystem timestamps to reconstruct a complete intrusion timeline.

---

## Methodology

### 1. Prefetch analysis
Navigated to C:\Windows\Prefetch and located the backdoor execution artefact:

**File found:** BACKDOOR.EXE-DDE90664.pf

| Field | Value |
|-------|-------|
| Creation time | 2026-06-06 8:37:33 AM |
| Last modified | 2026-06-06 8:37:33 AM |
| File size | 5295 bytes |

### 2. Registry persistence check
Checked both user and system run keys for persistence mechanisms:

```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

**Findings:** No malicious entries found. Only legitimate entries present:
- HKCU: OneDrive autostart
- HKLM: Windows Defender, VMware Tools

### 3. Downloads folder timeline
```powershell
Get-ChildItem "C:\Users\Leteh\Downloads" | Select-Object Name, CreationTime, LastWriteTime
```

---

## Intrusion Timeline

| Time | Event |
|------|-------|
| 8:26:39 AM | Wireshark downloaded (investigator tool) |
| 8:36:42 AM | backdoor.exe downloaded from Kali HTTP server |
| 8:37:05 AM | backdoor.exe executed by user |
| 8:37:33 AM | Windows created prefetch file confirming execution |

---

## Forensic Findings

### What was found
- Prefetch file proving backdoor.exe executed at 8:37 AM
- Complete download-to-execution timeline reconstructed from filesystem timestamps
- No registry persistence established by attacker

### What was not found
- No registry run keys added by attacker
- Backdoor would not survive a reboot

---

## Key forensic takeaways

Prefetch files are one of the most valuable artefacts in Windows forensics because:
- They survive even after the malware is deleted
- They record the exact first and last execution time
- They are created within seconds of execution
- They cannot be easily disabled by standard users

The 23-second gap between download (8:36:42) and execution (8:37:05) suggests the user manually ran the file rather than an automated execution - important context for determining whether the victim was complicit or deceived.

---

## Next steps
- Perform memory forensics using Volatility to find Meterpreter in RAM
- Analyse browser history for the download event
- Check Windows event logs for the full logon and process chain
