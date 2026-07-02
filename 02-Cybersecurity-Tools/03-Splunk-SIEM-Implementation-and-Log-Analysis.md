# Splunk SIEM: Implementation and Log Analysis

This document details the deployment, configuration, and analysis capabilities of Splunk Enterprise as a Security Information and Event Management (SIEM) platform. It covers architecture, log forwarding configurations, Search Processing Language (SPL), and practical queries used to detect attack telemetry.

---

## Part 1: Splunk Core Architecture

Splunk functions as a centralized engine that collects, indexes, and searches machine-generated data. Its architecture relies on three primary components:

1.  **Forwarder (Universal Forwarder - UF):** A lightweight agent installed on endpoints (e.g., Windows 10, Linux servers) that monitors log sources and forwards events to indexers. It consumes minimal CPU/RAM.
2.  **Indexer:** The processing and storage engine. It receives raw data, parses it into individual events, assigns timestamps, indexes the data (making it searchable), and writes it to disk.
3.  **Search Head:** The user interface. It handles user search queries, runs dashboards, and coordinates search execution across multiple indexers.

---

## Part 2: Endpoint Data Ingestion via `inputs.conf`

To ingest endpoint logs (like Windows Event Logs and Sysmon telemetry), the Universal Forwarder on the victim endpoint must be configured to monitor specific log channels. This is handled in the `inputs.conf` file.

### Configuration Path
`C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf`

### Configuration Example
In our attack simulation lab, we configured the following `inputs.conf` file on the Windows VM to forward Security and Sysmon logs to the Splunk Indexer:

```ini
[WinEventLog://Security]
disabled = 0
index = endpoint
sourcetype = XmlWinEventLog:Security

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = endpoint
sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```

*   **`[WinEventLog://...]`:** Specifies the Windows Event Log path to monitor.
*   **`disabled = 0`:** Activates the input monitor.
*   **`index = endpoint`:** Directs the forwarded logs to the dedicated `endpoint` index in Splunk.
*   **`sourcetype = ...`:** Formats the logs as XML event logs, making it easier for Splunk to parse them into searchable key-value fields.

---

## Part 3: Search Processing Language (SPL) Fundamentals

SPL is Splunk's query language, used to filter, transform, and visualize logs.

### Basic Command Reference

*   **`index` / `sourcetype`:** The primary filters used to specify search scopes. Always scope searches by index to ensure high performance.
    ```splunk
    index=endpoint sourcetype="XmlWinEventLog:*"
    ```
*   **`stats`:** Groups and counts fields. Essential for aggregating data.
    ```splunk
    # Count occurrences of each event code
    index=endpoint | stats count by EventCode
    ```
*   **`eval`:** Calculates new fields or transforms existing values.
    ```splunk
    # Create a field showing memory usage in MB
    index=perfmon | eval memory_mb = memory_bytes / 1024 / 1024
    ```
*   **`table`:** Filters columns to present data in a clean table format.
    ```splunk
    index=endpoint | table _time, EventCode, host, Image
    ```
*   **`dedup`:** Removes duplicate values based on specified fields.
    ```splunk
    # Get a list of unique active users
    index=endpoint | dedup User
    ```

---

## Part 4: Practical Incident Response Case Study

In our **SOC Home Lab — Attack Simulation & Detection**, we set up a Kali Linux VM (attacker) to establish a Meterpreter shell on a Windows 10 Pro VM (victim). Splunk successfully captured **3,723 events** during the simulation. Here are the search queries used to analyze the telemetry:

### 1. Identifying Process Execution (Sysmon Event ID 1)
To locate the execution of the malicious reverse shell payload (`msfvenom` generated binary):
```splunk
index=endpoint EventCode=1 
| table _time, parent_process_name, process_name, CommandLine, User
```
*   *Telemetry Result:* Identified `powershell.exe` spawning from anomalous parent binaries or running obfuscated base64 payloads.

### 2. Hunting for Network Connections (Sysmon Event ID 3)
To find out where the reverse shell payload connected back to (the attacker's C2 listener):
```splunk
index=endpoint EventCode=3 SourceIp=192.168.20.12
| table _time, Image, SourceIp, SourcePort, DestinationIp, DestinationPort
```
*   *Telemetry Result:* Identified the connection outbound from the victim (`192.168.20.12`) to the Kali VM (`192.168.20.10`) on port `4444` (the Metasploit port).

### 3. Detecting Registry Tampering for Persistence (Sysmon Event ID 13)
To track if the attacker added a registry run key for boot persistence:
```splunk
index=endpoint EventCode=13 RegistryEventType="SetValue" 
| table _time, process_name, TargetObject, Details
```
*   *Telemetry Result:* Monitored modifications under `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`.
