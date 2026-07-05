# Cybersecurity Tools

This folder collects my practical notes on tools I have used while learning networking, SOC analysis, detection engineering, incident response, web security, and lab-based offensive security.

The goal is not just to list commands. For each tool, I try to capture:

- What problem the tool solves
- Where it fits in a security workflow
- What an analyst should look for
- What kind of evidence or output the tool produces
- How I used or would use it in a safe lab environment

## Tool Categories

| Category | Tools | Main Use Cases |
| --- | --- | --- |
| Packet analysis | Wireshark | PCAP review, protocol analysis, traffic troubleshooting |
| Reconnaissance | Nmap | Port scanning, service discovery, network mapping |
| Network detection | Snort, Zeek | IDS alerts, protocol logs, network security monitoring |
| SIEM | Splunk, Wazuh | Log collection, searching, correlation, alerting |
| EDR and endpoint telemetry | Aurora, Wazuh Agent, Sysmon | Process monitoring, detection rules, endpoint investigation |
| Incident response | TheHive | Case management, alert triage, analyst workflow |
| Web testing | Burp Suite | Request interception, vulnerability testing, web app analysis |
| Lab exploitation | Metasploit | Controlled attack simulation and detection validation |

## Notes in This Folder

- [Wireshark Practical Traffic Analysis](./01-Wireshark-Practical-Traffic-Analysis.md)
- [Wireshark Advanced Analysis and Local Traffic](./02-Wireshark-Advanced-Analysis-and-Local-Traffic.md)
- [Nmap Reconnaissance and Enumeration](./03-Nmap-Reconnaissance-and-Enumeration.md)
- [Snort and Zeek Network Detection](./04-Snort-and-Zeek-Network-Detection.md)
- [SIEM Tools: Splunk and Wazuh](./05-SIEM-Splunk-and-Wazuh.md)
- [EDR Tools: Aurora and Wazuh Agent](./06-EDR-Aurora-and-Wazuh-Agent.md)
- [TheHive Incident Response Platform](./07-TheHive-Incident-Response.md)
- [Burp Suite Web Application Testing](./08-Burp-Suite-Web-Testing.md)
- [Metasploit for Lab Attack Simulation](./09-Metasploit-Lab-Attack-Simulation.md)
- [Tool Selection Cheat Sheet](./10-Tool-Selection-Cheat-Sheet.md)

## How These Tools Connect in a SOC Workflow

```text
Endpoint Activity
      |
      v
Sysmon / Wazuh Agent / Aurora
      |
      v
SIEM: Splunk or Wazuh
      |
      v
Detection Rule / Alert
      |
      v
TheHive Case
      |
      v
Analyst Investigation
```

Network tools fit beside this flow:

```text
Network Traffic
      |
      v
Wireshark / Zeek / Snort
      |
      v
PCAPs, protocol logs, IDS alerts
      |
      v
SIEM correlation and incident response
```

## Lab Safety Notes

Tools such as Burp Suite, Nmap, and Metasploit should only be used against systems I own, lab machines, intentionally vulnerable targets, or platforms where testing is explicitly authorized. My goal with these tools is to understand attack behavior so I can detect, investigate, and respond to it better.
