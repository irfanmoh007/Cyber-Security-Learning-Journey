# Metasploit for Lab Attack Simulation

## What Metasploit Is

Metasploit is an exploitation and post-exploitation framework. In my learning journey, its main value is controlled lab simulation: generating realistic attacker behavior so I can observe, detect, and investigate it from the defender side.

Metasploit should only be used in isolated labs, owned systems, CTF rooms, or environments with explicit authorization.

## Defensive Learning Use Cases

| Use Case | Defensive Value |
| --- | --- |
| Simulate a payload execution | See process, file, and network telemetry |
| Generate reverse shell behavior | Learn what C2-like traffic looks like |
| Validate SIEM detections | Confirm rules trigger on realistic activity |
| Practice incident response | Build a timeline from endpoint and network logs |
| Compare telemetry sources | See what Sysmon, Splunk, Wazuh, and EDR each capture |

## Evidence Metasploit Activity Can Create

| Evidence Source | Possible Signal |
| --- | --- |
| Sysmon Event ID 1 | Payload process creation |
| Sysmon Event ID 3 | Outbound connection to handler |
| Windows Security logs | Logon or process-related activity |
| EDR alerts | Suspicious tool, behavior, or command line |
| SIEM | Correlated process and network activity |
| Firewall logs | Outbound connection to listener IP/port |
| Zeek conn.log | Connection metadata |

## Questions for a SOC Investigation

- What file was executed?
- Which user executed it?
- What was the parent process?
- Did it connect outbound?
- What destination IP and port were used?
- Was the destination internal or external?
- Did the process spawn a shell?
- What commands followed execution?
- Which detections fired and which did not?

## Lab Detection Ideas

Potential detections to test in a home lab:

- Executable with double extension such as `.pdf.exe`
- Process connects to an unusual outbound port
- Suspicious child process from a user-downloaded file
- Command shell launched from an unexpected parent
- Known offensive tool names or metadata
- Reverse shell style network behavior

## Why This Tool Matters for Blue Team Learning

Reading about attacks is useful, but seeing the telemetry is better. Metasploit gives me a controlled way to create attack-like behavior and then answer the real SOC question: "Would I have detected this?"
