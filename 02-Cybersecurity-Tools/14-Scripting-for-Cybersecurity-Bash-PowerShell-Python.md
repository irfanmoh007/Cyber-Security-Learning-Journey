# Scripting for Cybersecurity: Bash, PowerShell, & Python

This document details the practical application of scripting in cybersecurity. It covers core commands, commandlets, and libraries used in **Bash**, **PowerShell**, and **Python** for log analysis, threat hunting, and security automation.

---

## Part 1: Why Scripting Matters in Security

In modern security operations, analyzing data manually is impossible due to the sheer volume of logs. Scripting enables security analysts to:
1.  **Parse and Triage Logs:** Filter through gigabytes of raw data to find specific indicators (IPs, hashes).
2.  **Automate Investigations:** Retrieve threat intelligence from APIs, query active network connections, and scan directories automatically.
3.  **Perform Incident Response:** Gather memory artifacts, stop malicious processes, and check system configurations on compromised endpoints.

---

## Part 2: Bash for Linux Log Analysis

Linux hosts generate system telemetry in plain text files (e.g., `/var/log/auth.log`, web server access logs). Analysts use utility commands to filter and format this data.

### Essential Utilities
*   **`grep`:** Filters lines containing specific search terms.
*   **`awk` / `cut`:** Extracts specific columns of text from structured files.
*   **`sort` / `uniq`:** Sorts lines and removes duplicates (or counts occurrences).

### Real-World Example: Parsing Apache Access Logs
To identify the top 5 IP addresses performing the most requests on a web server:
```bash
# Read log -> Extract IP column (column 1) -> Sort -> Count unique occurrences -> Sort by count -> Show top 5
cat /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 5
```
*   *Output:* A clean table showing the IP address and the exact number of requests made, allowing the analyst to quickly identify potential brute-forcing or scanning activity.

---

## Part 3: PowerShell for Windows Threat Hunting

PowerShell is the native shell and scripting environment for Windows systems administration. For a defender, it is the primary tool for querying endpoint states and Windows Event Logs.

### Critical Cmdlets
*   **`Get-Process`:** Lists active processes (equivalent to Task Manager).
*   **`Get-NetTCPConnection`:** Shows active network connections (equivalent to netstat).
*   **`Get-WinEvent`:** Queries the Windows Event Log database directly.

### Real-World Example: Finding Hidden Network Connections
To list all active network connections initiated by common target binaries (like `cmd.exe` or `powershell.exe`):
```powershell
Get-NetTCPConnection -State Established | 
Where-Object { (Get-Process -Id $_.OwningProcess).Name -match "cmd|powershell|wscript" } | 
Select-Object LocalAddress, LocalPort, RemoteAddress, RemotePort, State, @{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess).Name}}
```
*   *Security Value:* Instantly flags any reverse shells spawned from system command lines, helping analysts detect active C2 channels.

---

## Part 4: Python for Security Automation and Tool Integration

Python is the standard language for connecting security tools, querying APIs, and writing custom automation scripts.

### Essential Security Libraries
*   **`requests`:** Handles HTTP requests (used to query APIs like VirusTotal, Shodan, or AlienVault).
*   **`json`:** Parses JSON payloads returned by APIs.
*   **`re`:** Executes Regular Expressions to find patterns (like extraction of IP addresses).

### Real-World Example: Automated API Enrichment Script
The following Python script takes an IP address, queries the AlienVault OTX (Open Threat Exchange) reputation API, and prints whether it is flagged as malicious:

```python
import requests
import json
import sys

def check_ip_reputation(ip):
    # Public API endpoint for AlienVault OTX
    url = f"https://otx.alienvault.com/api/v1/indicators/IPv4/{ip}/general"
    
    headers = {"User-Agent": "Security Analyst Script"}
    
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            data = response.json()
            # Extract threat indicators count
            pulse_count = data.get("pulse_info", {}).get("count", 0)
            if pulse_count > 0:
                print(f"[!] Warning: {ip} is flagged in {pulse_count} threat intel pulses!")
            else:
                print(f"[+] {ip} is clean (0 active pulses).")
        else:
            print(f"[-] Error: API returned status code {response.status_code}")
    except Exception as e:
        print(f"[-] Connection failed: {e}")

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python check_ip.py [IP_ADDRESS]")
        sys.exit(1)
    check_ip_reputation(sys.argv[1])
```
*   *Security Value:* This script can be run locally or integrated into a SOAR workflow to automatically triage IP indicators found in firewall logs.
