# Mohamed Irfan's Cybersecurity Learning Journey

Welcome to my public learning repository! This project serves as a detailed chronicle of my journey into the field of cybersecurity. Here, I document concepts, create summaries, and share notes as I progress from zero knowledge to more advanced security topics.

## 📂 Repository Structure

The repository is organized hierarchically into topic-specific folders and hands-on projects:

```
Cyber-Security-Learning-Journey/
├── 01-Networking/                  # Networking Fundamentals
├── 02-Cybersecurity-Tools/          # Traffic analysis and security tools
├── 03-Linux/                        # Linux foundations and forensics
├── 04-TryHackMe-Rooms/              # TryHackMe writeups and lab notes
└── 05-Projects/                     # Hands-on security lab projects
```

---

## 📖 Table of Contents

### 🌐 Topic 01: Networking Fundamentals
* [**01: Core Concepts, Protocols, and Addressing**](./01-Networking/01-Core-Concepts.md)
* [**02: Protocols Deep Dive (ICMP, DNS, HTTP/HTTPS)**](./01-Networking/02-Protocols-Deep-Dive.md)
* [**03: Common Protocols & Examples**](./01-Networking/03-Common-Protocols-and-Examples.md)
* [**04: VPNs & Wireless Security**](./01-Networking/04-VPNs-and-Wireless-Security.md)
* [**05: Consolidated Foundations**](./01-Networking/05-Networking-Foundations-Consolidated.md) - *A comprehensive guide combining key ideas from earlier documents.*

### 🛠️ Topic 02: Cybersecurity Tools
* [**01: Practical Traffic Analysis with Wireshark**](./02-Cybersecurity-Tools/01-Wireshark-Practical-Traffic-Analysis.md) - *Dissection and analysis of network packets.*
* [**02: Advanced Traffic Analysis and Local Traffic**](./02-Cybersecurity-Tools/02-Wireshark-Advanced-Analysis-and-Local-Traffic.md) - *Deep dive into advanced Wireshark traffic analysis.*
* [**03: Splunk SIEM: Implementation and Log Analysis**](./02-Cybersecurity-Tools/03-Splunk-SIEM-Implementation-and-Log-Analysis.md) - *Enterprise SIEM deployment, inputs.conf, and SPL query analysis.*
* [**04: Wazuh SIEM & XDR Operations**](./02-Cybersecurity-Tools/04-Wazuh-SIEM-and-XDR-Operations.md) - *Agent-manager architecture, File Integrity Monitoring (FIM), and custom detection rules.*
* [**05: Windows System Monitor (Sysmon)**](./02-Cybersecurity-Tools/05-Sysmon-Windows-System-Monitor.md) - *Low-level process execution, connection, registry, and DNS telemetry collection.*
* [**06: Shuffle SOAR and Incident Automation**](./02-Cybersecurity-Tools/06-Shuffle-SOAR-and-Incident-Automation.md) - *Security Orchestration, Automation, and Response playbooks, webhook ingestion, and VirusTotal enrichment.*
* [**07: TheHive: Incident Response Platform**](./02-Cybersecurity-Tools/07-TheHive-Incident-Response-Platform.md) - *Ticketing, case management, tracking observables, and NIST incident response lifecycle.*
* [**08: Nmap and Network Reconnaissance**](./02-Cybersecurity-Tools/08-Nmap-and-Network-Reconnaissance.md) - *Active/passive OSINT, TCP SYN scans, UDP port scanning, and NSE scripts.*
* [**09: Metasploit and MSFvenom Exploitation**](./02-Cybersecurity-Tools/09-Metasploit-and-MSFvenom-Exploitation.md) - *Attack simulation, payload creation (staged/non-staged), handlers, and Meterpreter post-exploitation.*
* [**10: Burp Suite: Web Application Penetration Testing**](./02-Cybersecurity-Tools/10-Burp-Suite-Web-Pentesting.md) - *Intercepting proxy settings, SSL/TLS CA certificate installation, and Repeater/Intruder attacks.*
* [**11: CyberChef: The Cyber Swiss Army Knife**](./02-Cybersecurity-Tools/11-CyberChef-The-Cyber-Swiss-Army-Knife.md) - *Data encoding/decoding, regex data parsing, hashing, and defanging threat intelligence indicators.*
* [**12: Cybersecurity Frameworks: MITRE ATT&CK & Cyber Kill Chain**](./02-Cybersecurity-Tools/12-Cybersecurity-Frameworks-MITRE-ATT-CK-and-Kill-Chain.md) - *Lockheed Martin Kill Chain, MITRE ATT&CK Enterprise TTPs, and threat intelligence.*
* [**13: Active Directory and Enterprise Security**](./02-Cybersecurity-Tools/13-Active-Directory-and-Enterprise-Security.md) - *Kerberos ticket-granting workflow, AD directory protocols, common attacks (Kerberoasting, PtH, Golden Ticket), and auditing logs.*
* [**14: Scripting for Cybersecurity: Bash, PowerShell, & Python**](./02-Cybersecurity-Tools/14-Scripting-for-Cybersecurity-Bash-PowerShell-Python.md) - *Utility commands for log parsing, PowerShell host forensics, and custom Python API scripts.*
* [**15: Aurora EDR and Endpoint Detection**](./02-Cybersecurity-Tools/15-Aurora-EDR-and-Endpoint-Detection.md) - *Lightweight agent, ETW process/memory tracing, and Sigma rule evaluation on endpoints.*


### 🐧 Topic 03: Linux Foundations
* [**01: Linux Commands**](./03-Linux/01-Linux-Commands.md) - *A reference list of standard Linux navigation, manipulation, and permission commands.*
* [**02: Linux Forensics Cheatsheet**](./03-Linux/02-Linux-Forensics-Cheatsheet.md) - *Quick reference for forensic examination, OS information, log locations, and persistence mechanisms.*

### 🎮 Topic 04: TryHackMe Room Writeups
Detailed writeups and walk-throughs for various TryHackMe (THM) security challenges:
* [**Aurora EDR**](./04-TryHackMe-Rooms/Aurora-EDR.md) - *Working with Aurora EDR detection and log analysis.*
* [**Boogeyman 1**](./04-TryHackMe-Rooms/Boogeyman-1.md) - *Investigating phishing emails and macro-enabled documents.*
* [**Boogeyman 2**](./04-TryHackMe-Rooms/Boogeyman-2.md) - *Analyzing system execution and persistence of the adversary.*
* [**Boogeyman 3**](./04-TryHackMe-Rooms/Boogeyman-3.md) - *Completing the threat hunting and forensic investigation of an attack.*
* [**Fix It**](./04-TryHackMe-Rooms/Fix-It.md) - *Debugging and fixing broken logging configurations and permissions.*
* [**Incident Response Preparation**](./04-TryHackMe-Rooms/Incident-Response-Preparation.md) - *Understanding and setting up key phases of incident response.*
* [**Infinity Shell**](./04-TryHackMe-Rooms/Infinity-Shell.md) - *Exploiting Web Shells and understanding remote shell mechanics.*
* [**Investigating Windows**](./04-TryHackMe-Rooms/Investigating-Windows.md) - *Analyzing registry, event logs, and user activity on a compromised Windows host.*
* [**Juicy Details**](./04-TryHackMe-Rooms/Juicy-Details.md) - *Web application security, information disclosure, and forensics.*
* [**Setup and Configuring SOC Config Files**](./04-TryHackMe-Rooms/Setup-and-Configuring-SOC-Config-Files.md) - *Configuring monitoring infrastructure logs and tools.*
* [**SigHunt**](./04-TryHackMe-Rooms/SigHunt.md) - *Writing and deploying YARA rules to detect malware signatures.*
* [**Sneaky Patch**](./04-TryHackMe-Rooms/Sneaky-Patch.md) - *Binary analysis and patching techniques.*
* [**Volatility Room**](./04-TryHackMe-Rooms/Volatility-Room.md) - *Memory forensics and analysis using the Volatility tool.*
* [**Zeek**](./04-TryHackMe-Rooms/Zeek.md) - *Network security monitoring and log parsing using Zeek.*

### 🧪 Topic 05: Hands-on Projects
* [**SOC Basic Home Lab**](./05-Projects/SOC-Basic-HomeLab/SOC-Basic-HomeLab.md) - *Building a basic Security Operations Center (SOC) home lab with Splunk, VirtualBox, and Metasploit, simulating attacks and analyzing logs.*
* [**SOC Automation Lab**](./05-Projects/SOC-Automation-Lab/README.md) - *Setting up automated logging, EDR, SIEM, SOAR, and Incident response orchestration using Wazuh, Shuffle, and TheHive.*
