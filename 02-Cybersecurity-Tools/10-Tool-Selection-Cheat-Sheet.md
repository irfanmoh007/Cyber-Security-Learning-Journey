# Tool Selection Cheat Sheet

This note helps me choose the right tool for the investigation question.

## Quick Tool Map

| Question | Tool to Start With |
| --- | --- |
| What hosts and ports exist? | Nmap |
| What is inside this packet capture? | Wireshark |
| What connections happened on the network? | Zeek |
| Did traffic match a known IDS signature? | Snort |
| What happened on this endpoint? | Sysmon, Wazuh Agent, Aurora |
| Can I search all logs for this host or user? | Splunk or Wazuh |
| Did this alert become an incident? | TheHive |
| What did this web request actually send? | Burp Suite |
| Can I safely simulate attacker behavior in a lab? | Metasploit |

## Investigation Flow by Scenario

### Suspicious Process on Windows

Start with:

- Sysmon Event ID 1
- Wazuh or Splunk search
- Aurora/EDR alert details

Then pivot to:

- Parent process
- Command line
- File path
- User account
- Network connections
- Related hashes or domains

### Suspicious Outbound Network Connection

Start with:

- Firewall logs
- Zeek `conn.log`
- Sysmon Event ID 3
- SIEM search by destination IP/port

Then pivot to:

- Process responsible for the connection
- DNS query before the connection
- TLS SNI or certificate metadata
- Other hosts contacting the same destination

### Web Application Suspicion

Start with:

- Burp Suite for request understanding in a lab
- Web server logs in real investigations
- SIEM search for IP, user, path, status code, and user agent

Then pivot to:

- Repeated object ID changes
- Login/reset behavior
- Upload attempts
- Admin routes
- Error responses

### Possible Malware Execution

Start with:

- EDR or Wazuh alert
- Sysmon process creation
- File creation logs
- Hash enrichment if available

Then pivot to:

- Command line
- Parent-child process tree
- Persistence changes
- Network connections
- TheHive case documentation

## Tool Strengths

| Tool | Strength |
| --- | --- |
| Wireshark | Deep packet-level understanding |
| Nmap | Fast network and service discovery |
| Snort | Signature-based IDS alerting |
| Zeek | Rich network metadata and hunting logs |
| Splunk | Flexible log search and dashboards |
| Wazuh | Open-source endpoint monitoring and SIEM workflow |
| Aurora | Endpoint detection and threat hunting logic |
| TheHive | Incident case management |
| Burp Suite | Web request inspection and testing |
| Metasploit | Lab-based attack simulation for detection validation |

## My Main Takeaway

No single tool tells the whole story. Good analysis comes from pivoting between endpoint evidence, network evidence, SIEM searches, and case notes until the timeline makes sense.
