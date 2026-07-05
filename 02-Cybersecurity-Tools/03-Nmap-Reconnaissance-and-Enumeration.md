# Nmap Reconnaissance and Enumeration

## What Nmap Is

Nmap is a network discovery and service enumeration tool. It helps answer basic but important questions:

- Which hosts are alive?
- Which ports are open?
- What services are running?
- What versions might those services be?
- What operating system or device type might be present?

For defenders, Nmap is useful because attackers often begin with similar discovery steps. Understanding Nmap output makes it easier to recognize exposed services, unnecessary attack surface, and suspicious scanning behavior in logs.

## Common Use Cases

| Use Case | Why It Matters |
| --- | --- |
| Asset discovery | Identify live hosts on a subnet |
| Port scanning | Find exposed services such as SSH, HTTP, SMB, RDP, or databases |
| Service enumeration | Determine application names and versions |
| Vulnerability validation | Confirm whether a risky service is actually exposed |
| Lab recon | Build an attack path during authorized practice rooms |
| Defensive baselining | Compare expected open ports against actual open ports |

## Scan Types I Should Understand

| Scan Type | Purpose | Analyst Note |
| --- | --- | --- |
| Ping sweep | Find live hosts | May be blocked by firewalls |
| TCP SYN scan | Identify open TCP ports | Common and fast |
| TCP connect scan | Full TCP connection | Easier to log on targets |
| UDP scan | Check UDP services | Slower but important for DNS, SNMP, NTP |
| Version detection | Identify service versions | Useful for risk assessment |
| OS detection | Guess operating system | Not always accurate |
| Script scan | Run NSE scripts | Powerful, but use carefully and only with authorization |

## Example Defensive Questions

When reviewing Nmap results, I should ask:

- Is this service expected on this host?
- Is the port exposed to the right network segment?
- Is the service version outdated?
- Is management access exposed, such as SSH, RDP, WinRM, or admin panels?
- Does the host expose file sharing or database services unnecessarily?
- Would a SIEM alert if this scan occurred in a production network?

## Useful Output Fields

| Field | Meaning |
| --- | --- |
| Host | IP address or hostname being scanned |
| Port | Network port number |
| State | Open, closed, filtered, or unknown |
| Service | Guessed service name |
| Version | Detected product/version, if available |
| Reason | Why Nmap believes the port has that state |

## Detection Opportunities

Nmap activity can create visible patterns:

- Many connection attempts from one source to many ports
- Sequential port scanning
- SYN packets without full application use
- Failed connections across many hosts
- DNS or reverse lookup activity during recon
- Web server logs showing unusual probes

In a SIEM, this can become a rule such as:

```text
Same source IP connects to many destination ports in a short time window.
```

## How Nmap Fits My Learning Path

Nmap connects networking knowledge with security practice. To use it well, I need to understand ports, TCP flags, services, firewall behavior, and how scans appear from the defender side.
