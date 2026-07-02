# Windows System Monitor (Sysmon)

This document provides a technical guide to Microsoft Sysinternals System Monitor (Sysmon). It covers installation, modular configuration approaches, critical Event IDs, and how to analyze Sysmon events to detect malicious activity on Windows endpoints.

---

## Part 1: What is Sysmon?

Sysmon is a Windows system service and device driver that monitors and logs system activity to the Windows Event Log. Unlike standard Windows Security logs, which primarily track authentications and policy changes, Sysmon captures deep, low-level system telemetry such as process creations, network connections, file modifications, and registry changes.

Sysmon remains resident across system reboots, collecting events that are essential for security analysts to construct a chronological timeline of an intrusion.

---

## Part 2: Installation and Configurations

### 1. Installation
Sysmon is installed via the command line. It requires administrative privileges and a configuration file (in XML format) that defines what system events should be filtered in or out.

```powershell
# Install Sysmon (64-bit) and accept the EULA, using a custom config file
sysmon64.exe -accepteula -i sysmonconfig.xml
```

*   **`-i`:** Installs the service and driver.
*   **`-c`:** Updates an existing Sysmon configuration.
*   **`-u`:** Uninstalls the service and driver.

### 2. Configuration Strategy
By default, Sysmon logs *everything*, which generates massive log volumes (noise). To prevent clogging system resources and SIEM licenses, analysts use modular configurations.
*   **SwiftOnSecurity Config:** Focuses on high-fidelity security events and filters out benign Windows background noise (e.g., standard OS DLL loads). Highly recommended for general endpoint coverage.
*   **Olaf Hartong (Sysmon-modular) Config:** A highly structured, modular configuration mapped directly to MITRE ATT&CK techniques, allowing precise categorization of security alerts.

---

## Part 3: Critical Event IDs Reference

Sysmon uses Event IDs to categorize different system actions. The most critical Event IDs for security analysis are:

### Event ID 1: Process Creation
Logs when a new process starts. It includes:
*   `CommandLine`: The exact command executed (crucial for finding obfuscated scripting payloads).
*   `ParentCommandLine`: The command that spawned this process.
*   `Hashes`: Cryptographic hashes (SHA-256, MD5) of the executable file, useful for matching threat intelligence indicators.

### Event ID 3: Network Connection
Logs TCP/IP connections established on the host. It includes:
*   `SourceIp` and `DestinationIp`: Identifies internal victims and remote C2 servers.
*   `DestinationPort`: Detects anomalous ports (e.g., a Microsoft Word process initiating an outbound connection on port `4444`).

### Event ID 11: File Create
Logs when a file is created or overwritten. This is critical for detecting payload downloads or temporary files created by malware:
*   `TargetFilename`: The path where the new file was created.

### Event ID 12, 13, 14: Registry Events
Logs registry key and value modifications. Registry tampering is a primary indicator of persistence:
*   `Event ID 12`: Registry key create and delete.
*   `Event ID 13`: Registry value set (e.g., adding a binary path to the `Run` key).
*   `Event ID 14`: Registry key and value rename.

### Event ID 22: DNS Query
Logs when a process performs a DNS query. This is essential for tracking C2 domain lookups:
*   `QueryName`: The domain name requested (e.g., resolving `attacker-site.com`).

---

## Part 4: SIEM Integration in Projects

In our SOC laboratories, Sysmon was the primary engine used to gather endpoint telemetry:

1.  **SOC Basic HomeLab:** Ingested Sysmon events via Splunk Universal Forwarder. Using **Event ID 1** and **Event ID 3**, we traced how a custom executable compiled with `msfvenom` spawned a process and initiated an outbound connection back to a Kali Linux handler.
2.  **SOC Automation Lab:** Installed Sysmon on a Windows 11 endpoint with the Olaf Hartong configuration. The Wazuh Agent forwarded these events to the Wazuh Manager, which matched a Sysmon process creation event containing `mimikatz.exe` to trigger a critical alert.
