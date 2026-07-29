# EDR Tools: Aurora and Wazuh Agent

## What EDR Means

Endpoint Detection and Response tools monitor activity on endpoints such as laptops, desktops, and servers. They help analysts detect suspicious process behavior, file activity, persistence, credential access, and other host-level events.

Endpoint telemetry is critical because many attacks become visible first on the host:

- A suspicious process starts
- PowerShell runs with unusual arguments
- A file is written to a strange location
- A process creates a network connection
- A registry key changes
- A credential dumping tool executes

## Aurora

Aurora is a detection-focused endpoint tool from Nextron Systems. It is useful for learning how endpoint detections are written and how suspicious activity appears in logs.

### Aurora Use Cases

| Use Case | Why It Matters |
| --- | --- |
| Endpoint detection | Identify suspicious files, commands, and behaviors |
| Threat hunting | Search for indicators and suspicious patterns |
| Rule learning | Understand how Sigma/YARA-style logic maps to host events |
| Lab validation | Test whether a simulated behavior is detected |

### What to Look For

- Suspicious command-line arguments
- Known attacker tools or filenames
- Unusual parent-child process relationships
- File creation in user-writable paths
- Encoded or obfuscated PowerShell
- Persistence locations
- Credential access behavior

## Wazuh Agent

The Wazuh Agent collects endpoint logs and sends them to the Wazuh Manager. It is especially useful when combined with Sysmon on Windows.

### Wazuh Agent Use Cases

| Use Case | Example |
| --- | --- |
| Log forwarding | Send Windows, Linux, and Sysmon events to Wazuh |
| File integrity monitoring | Detect changes to important files |
| Command monitoring | Track selected system commands |
| Security configuration checks | Identify weak or risky settings |
| Custom detections | Trigger Wazuh rules from endpoint events |

## Sysmon as Endpoint Telemetry

Sysmon is not an EDR by itself, but it provides high-value Windows telemetry that EDR and SIEM workflows depend on.

Important Sysmon events:

| Event ID | Meaning |
| --- | --- |
| 1 | Process creation |
| 3 | Network connection |
| 7 | Image loaded |
| 10 | Process access |
| 11 | File creation |
| 13 | Registry value set |
| 22 | DNS query |

## Endpoint Investigation Questions

When reviewing EDR or endpoint logs, I should ask:

- What process started the suspicious activity?
- What was the parent process?
- What command line was used?
- Which user account ran it?
- Did it create files?
- Did it connect to the network?
- Did it modify persistence locations?
- Are there related alerts from SIEM, Zeek, Snort, or firewall logs?

## Why EDR Matters

Network logs can show that a connection happened, but endpoint logs often explain why it happened. For SOC work, endpoint telemetry is one of the strongest sources for reconstructing an attack chain.
