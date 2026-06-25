# TryHackMe — Threat Hunting: Foothold

## Overview

This room teaches threat hunting through the lens of the **MITRE ATT&CK** kill chain — not by giving you a single IOC to find, but by walking an entire intrusion from the attacker's first knock on the door to their final C2 channel. The investigation happens in the **Elastic Stack (Kibana)**, using two main log sources:

- `packetbeat-*` — network traffic (DNS, HTTP, raw connections)
- `winlogbeat-*` — Windows event logs / Sysmon (process creation, registry, network correlation)

The core skill this room builds is **pivoting between network and host logs** — see something weird on the wire, then jump to the endpoint logs to find out *which process* caused it, and vice versa.

**The room's mental model, end to end:**

```
Initial Access → Execution → Defense Evasion → Persistence → Command & Control
```

This mirrors a real adversary's actual order of operations, and the room structures the hunt the same way.

---

## The Threat Hunter's Mindset (the actual "trick" of this room)

Every section in this room follows the same loop:

1. **Form a hypothesis** based on a tactic (e.g. "if there's C2 over DNS, I'd expect to see one domain getting hammered with weird subdomains").
2. **Query broadly first** (visualize/aggregate) to spot the outlier — don't go log-diving blind.
3. **Narrow the query** once you've got a lead (a domain, an IP, a process name).
4. **Pivot indices** — go from `packetbeat-*` (network) to `winlogbeat-*` (host) to find *the process responsible*.
5. **Use "View surrounding documents"** to walk backward/forward in time around a suspicious event — this is how you find the parent process, or the command that came right before/after.
6. **Ask "what would the attacker do next?"** — every section ends with "the next step is to identify X," which is the room nudging you to keep pivoting instead of stopping at one IOC.

If you remember nothing else from this room, remember this loop. It's the actual transferable skill — the specific KQL queries below are just this loop applied to five different ATT&CK tactics.

---

## 1. Initial Access

**Goal:** how did the attacker first get in? Every technique (phishing, exploiting a vuln, credential spraying, malicious USB, cracked software) boils down to one of two outcomes:

- Valid account access (stolen/brute-forced credentials)
- Remote code execution (malicious file gets run)

### 1a. SSH Brute Force

Hypothesis: if there's a brute force, there'll be a pile of failed auth attempts from one source.

```
host.name: jumphost AND event.category: authentication AND system.auth.ssh.event: Failed
```

Visualize this as a **Lens table** to spot which source IP is generating the bulk of failures — that's your suspect IP.

Once you have a suspect IP, check if they ever actually got in:

```
host.name: jumphost AND event.category: authentication AND system.auth.ssh.event: Accepted AND source.ip: (167.71.198.43 OR 218.92.0.115)
```

If there's an `Accepted` event for that IP, the brute force succeeded.

### 1b. Did they pivot to a web app?

Once credentials/access is confirmed, check if that IP touched anything else (e.g. an internal web server):

```
host.name: web01 AND network.protocol: http AND destination.port: 80 AND source.ip: 167.71.198.43 AND http.response.status_code: (200 OR 301 OR 302)
```

A 200/301/302 means the request actually succeeded — not just attempted. Add `url.query` and `user_agent.original` as columns to see *what* they accessed.

### 1c. Phishing (link/attachment based)

Hypothesis: phishing → user downloads something via browser or opens an attachment via email client → that file gets executed.

**Step 1 — did Chrome download anything?**
```
host.name: WKSTN-* AND process.name: chrome.exe AND winlog.event_id: 11
```
(Event ID 11 = Sysmon "FileCreate".)

Columns to add: `winlog.computer_name`, `winlog.event_data.User`, `file.name` (target/file path).

**Step 2 — did Outlook open a malicious attachment?**
```
host.name: WKSTN-* AND process.name: OUTLOOK.EXE AND winlog.event_id: 11
```

In this room's scenario, this surfaces a file called `Update.zip` saved to a temp directory.

**Step 3 — confirm what's inside/related to that zip:**
```
host.name: WKSTN-* AND *Update.zip*
```

This reveals an `.lnk` (Windows shortcut) file archived inside the zip — **`.lnk` inside a `.zip` is a textbook phishing payload pattern**, because `.lnk` files can run arbitrary commands but look like a harmless shortcut icon to the victim.

**Step 4 — what did the `.lnk` spawn?**
Use **View surrounding documents** on the `.lnk`-related event to see the process it triggered. This is the bridge from Initial Access into the next tactic, Execution.

> **Note to self:** I wasn't 100% sure why `.lnk` files specifically are abused — worth re-reading on this later. Short answer for future-me: `.lnk` shortcut files store a target command line in their metadata, so a "shortcut" can secretly point to `powershell.exe -enc <base64>` instead of a real file. Windows doesn't warn you the way it does for `.exe`.

---

## 2. Execution

**Goal:** how did the attacker get their code to actually run? Common patterns: command-line tool abuse (PowerShell/CMD), LOLBAS (Living Off the Land Binaries — abusing legitimate Windows tools), and scripting languages (Python/PHP/Node).

### 2a. Suspicious CMD/PowerShell usage

```
host.name: WKSTN-* AND winlog.event_id: 1 AND process.name: (cmd.exe OR powershell.exe)
```
(Event ID 1 = Sysmon "Process Create".)

Columns: `winlog.computer_name`, `user.name`, `process.parent.command_line`, `process.command_line`.

### 2b. PowerShell Script Block Logging (deeper visibility)

```
host.name: WKSTN-* AND winlog.event_id: 4104
```

Event ID 4104 logs the **actual script block content**, not just the launch command — this is how you see obfuscated/encoded PowerShell payloads in full. Filter out noise by removing `Set-StrictMode` events (the room says to exclude these via the minus/NOT button in Kibana — they're benign boilerplate that floods results).

Columns: `winlog.computer_name`, `winlog.user.name`, `powershell.file.script_block_text`.

**Red-flag strings to look for inside script blocks** (these aren't inherently malicious, but attackers use them constantly):
- `IEX` / `Invoke-Expression` — runs a string as code
- `-enc` — encoded (usually base64) command
- `-nop` / `-NoProfile` — skips loading PowerShell profile (faster, stealthier)
- `-w hidden` — hidden window
- `-ep bypass` / `ExecutionPolicy Bypass` — skips script execution restrictions
- `DownloadString` / `DownloadFile` — pulls a second-stage payload from the internet

### 2c. LOLBAS (Living Off the Land Binaries)

Attackers abuse legit Windows binaries to download/execute payloads while blending in. The room focuses on three classics: `mshta.exe`, `certutil.exe`, `regsvr32.exe`.

```
host.name: WKSTN-* AND winlog.event_id: (1 OR 3) AND (process.name: (mshta.exe OR certutil.exe OR regsvr32.exe) OR process.parent.name: (mshta.exe OR certutil.exe OR regsvr32.exe))
```

(Event ID 3 = Sysmon "Network Connection" — included because LOLBAS are often used to *reach out* to the internet, e.g. `certutil -urlcache -f`.)

Note: the query checks **both** `process.name` and `process.parent.name` so you catch the LOLBAS itself *and* anything it spawned as a child.

Columns: `winlog.computer_name`, `user.name`, `process.parent.command_line`, `process.name`, `process.command_line`, `destination.ip`.

### 2d. Scripting languages (Python/PHP/Node)

```
host.name: WKSTN-* AND winlog.event_id: (1 OR 3) AND (process.name: (*python* OR *php* OR *nodejs*) OR process.parent.name: (*python* OR *php* OR *nodejs*))
```

In this room's scenario: **Python spawns `cmd.exe`** and **connects out to `167.71.198.43:8080`** — this is the signature of a **Python reverse shell**.

**Pivoting via PID — an important technique:** once you spot Python spawning a `cmd.exe` child, grab that `cmd.exe`'s PID from the dropdown on the log entry, then hunt everything *that* process spawned:

```
host.name: WKSTN-* AND winlog.event_id: (1 OR 3) AND process.parent.pid: 1832
```

This confirmed a script called `dev.py` was acting as a reverse shell, letting the attacker run commands through `cmd.exe` remotely. The natural next step (room's hint, not yet investigated in this write-up): trace *backward* to find out how `dev.py` got written to disk in the first place.

---

## 3. Defense Evasion

**Goal:** how did the attacker avoid getting caught? Three big buckets: disabling security tooling, deleting logs, and process injection.

### 3a. Disabling Windows Defender

```
host.name: WKSTN-* AND (*DisableRealtimeMonitoring* OR *RemoveDefinitions*)
```

- `DisableRealtimeMonitoring` → used with PowerShell's `Set-MPPreference` to turn off real-time scanning.
- `RemoveDefinitions` → used with the built-in `MpCmdRun.exe` to wipe Defender's signature database.

Columns: `winlog.computer_name`, `user.name`, `process.parent.command_line`, `process.name`, `process.command_line`.

### 3b. Log Clearing

```
host.name: WKSTN-* AND winlog.event_id: 1102
```

Event ID 1102 = "The audit log was cleared." If this fires, **use View surrounding documents** to find the actual command that did the clearing (add `process.name` + `process.command_line` columns). In the room, this confirmed Windows Event Logs were wiped on `WKSTN-1` — a strong defense evasion signal, since it's usually one of the *last* things an attacker does before/after major activity, specifically to erase the trail.

> **Honest note:** the room doesn't fully explain *why* an attacker clears logs at this specific point vs. another — my own takeaway is that log clearing is typically used either right after a noisy action (to erase evidence of it) or as a parting move before going quiet. Worth digging into "anti-forensics" techniques further if revisiting this.

### 3c. Process Injection

```
host.name: WKSTN-* AND winlog.event_id: 8
```

Event ID 8 = Sysmon "CreateRemoteThread" — this fires when one process creates a thread inside *another* process's memory space (the actual mechanism of process injection).

Columns: `winlog.computer_name`, `process.executable`, `winlog.event_data.SourceUser`, `winlog.event_data.TargetImage`.

In the room: `C:\Users\clifford.miller\Downloads\chrome.exe` (note — this is **not the real Chrome**, just malware using a familiar name to blend in) injected a thread into `explorer.exe`. Injecting into `explorer.exe` is a very common technique because it's a trusted, always-running process — injected code "hides" inside something that looks completely normal.

Also notable: most entries here ran as **SYSTEM**, except the malicious `chrome.exe`, which ran under Clifford Miller's own user account — a useful detail for scoping "what does the attacker actually have access to right now."

---

## 4. Persistence

**Goal:** how does the attacker survive a reboot / stay in long-term? Big two: scheduled tasks and registry Run keys.

### 4a. Scheduled Tasks

```
host.name: WKSTN-* AND (winlog.event_id: 4698 OR (*schtasks* OR *Register-ScheduledTask*))
```

Event ID 4698 = "A scheduled task was created." Columns: `winlog.computer_name`, `user.name`, `process.command_line`, `winlog.event_id`, `winlog.event_data.TaskName`.

### 4b. Registry Run Keys

Broad query (Sysmon registry events):
```
host.name: WKSTN-* AND winlog.event_id: 13 AND winlog.channel: Microsoft-Windows-Sysmon/Operational
```
(Event ID 13 = Sysmon "RegistryEvent (Value Set)".)

**Problem:** this alone returned ~1481 results in the room — way too noisy to manually review ("needle in a haystack"). The fix is to **narrow by known abused registry paths**, since persistence-via-registry overwhelmingly clusters around a small handful of keys:

```
host.name: WKSTN-* AND winlog.event_id: 13 AND winlog.channel: Microsoft-Windows-Sysmon/Operational AND registry.path: (*CurrentVersion\\Run* OR *CurrentVersion\\Explorer\\User* OR *CurrentVersion\\Explorer\\Shell*)
```

Columns: `winlog.computer_name`, `user.name`, `process.name`, `registry.path`, `winlog.event_data.Details`.

This surfaced the actual finding:
- **Registry path:** `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx\0001\Depend\1`
- **Registry data:** `C:\Windows\Temp\installer.exe`

→ meaning `installer.exe` was configured to auto-run on startup. (Note: `installer.exe` is the same binary that shows up later under C2 over Discord — this is the room quietly threading one artifact through multiple tactics, which is realistic.)

**Alternative narrowing strategy — filter by the process doing the writing**, instead of by registry path:
```
host.name: WKSTN-* AND winlog.event_id: 13 AND winlog.channel: Microsoft-Windows-Sysmon/Operational AND process.name: (reg.exe OR powershell.exe)
```
Limitation (important): this only catches registry edits made *through* `reg.exe` or `powershell.exe`. A binary that talks to the registry API directly (no shell-out) won't show up here. The room's suggested workaround for that gap: instead of allow-listing suspicious tools, **exclude known-good/benign binaries** from a broad query — i.e., hunt by exception rather than by inclusion when you don't know what binary to expect.

---

## 5. Command and Control (C2)

**Goal:** how does the compromised machine "phone home"? Three flavors covered, all chosen by attackers because they blend in with normal traffic: **DNS, known cloud apps, and encrypted HTTP**.

### 5a. C2 over DNS (DNS Tunneling)

Hypothesis: a C2-over-DNS channel needs *many* unique subdomains under one domain (each subdomain encodes a chunk of exfiltrated/command data).

Start broad — visualize DNS request volume per domain:
```
network.protocol: dns AND NOT dns.question.name: *arpa
```
(`NOT *arpa` filters out reverse-DNS lookup noise, which is non-malicious background traffic.)

Build this as a **Lens table** with a "unique subdomain count" metric — the standout in the room: **`golge.xyz`** queried **2,191 unique subdomains** from `WKSTN-1`. That volume is the tell — legitimate domains don't get queried like that.

Drill into just that domain + host:
```
network.protocol: dns AND NOT dns.question.name: *arpa AND dns.question.registered_domain: golge.xyz AND host.name: WKSTN-1
```

What this revealed:
- Multiple **query types** in use (CNAME, TXT, MX) — tunneling tools often rotate record types to maximize data throughput / evade simple detection rules built around just "A record" queries.
- **Hexadecimal subdomains** — the actual encoded C2 data/commands, hidden as gibberish-looking subdomain labels.
- DNS requests sent **directly to an unknown nameserver**, bypassing the workstation's configured DNS server entirely — a major red flag, since normal DNS traffic goes through the configured resolver.

Pivot to find the **destination IP** of this traffic (visible in the network data), then jump to `winlogbeat-*` to find the **process** responsible:
```
host.name: WKSTN-1* AND destination.ip: 167.71.198.43 AND destination.port: 53
```

Columns: `host.name`, `user.name`, `process.parent.command_line`, `process.name`, `process.command_line`.

**Finding:** all of this traffic was generated by `nslookup.exe` — normally a benign diagnostic tool, here being scripted/automated by a parent process to perform tunneling. **Use View surrounding documents** to grab that parent process's full command line — this confirms the C2 channel and shows exactly how it was being driven.

**Bonus detection angle (the room flags this as a footnote, worth remembering):** check `network.bytes` / packet size. Legitimate DNS queries are short. A query carrying a long hex string in the subdomain (to smuggle data) will be noticeably larger than normal — packet size anomaly is a second, independent signal beyond subdomain volume.

### 5b. C2 over Cloud Apps (Discord)

Hypothesis: reuse the same C2-over-DNS visualization table, but flip the metric — remove "unique subdomain count" and **sort by count ascending** instead. This surfaces domains that are *rarely* visited — i.e., cloud services the workstation doesn't normally talk to.

This surfaced **`discord.gg`** being contacted by `WKSTN-1` — Discord is a legitimate, widely-trusted service, which is exactly why attackers abuse it (its traffic doesn't look suspicious to most network filters, and its CDN/webhook features make a convenient, free C2 relay).

Pivot to `winlogbeat-*` to find the process:
```
host.name: WKSTN-1* AND *discord.gg*
```

**Finding:** the connection is initiated by `C:\Windows\Temp\installer.exe` (same binary seen earlier in the Persistence section's RunOnceEx key — these threads are meant to connect).

Hunt everything this process spawned:
```
host.name: WKSTN-1* AND winlog.event_id: 1 AND process.parent.executable: "C:\\Windows\\Temp\\installer.exe"
```

**Finding:** `installer.exe` spawned **multiple `cmd.exe` instances** — confirming it's not just *connecting* to Discord, it's using that connection to receive and execute commands. The room's suggested next step: pull every event generated by `installer.exe` to map the full scope of what it actually did on the box.

### 5c. C2 over Encrypted HTTP Traffic

This is the "classic" C2 type — the attacker runs their own domain and encrypts/obfuscates the traffic over plain HTTP, rather than piggybacking on DNS or a trusted cloud app. What to hunt for:
- High count of HTTP requests to one distinctive domain
- High outbound bandwidth to a unique domain

Build a Lens table:
- **Index:** `packetbeat-*`
- **Rows:** `host.name`, `destination.domain`, `http.request.method`
- **Metric:** count

Query:
```
network.protocol: http AND network.direction: egress
```

**Finding:** `cdn.golge.xyz` (note: same root domain family as the DNS tunneling C2 — same attacker infrastructure, different channel) showed a high volume of outbound connections from **both** workstations — suggesting a long-running, continuous C2 session.

Narrow further:
```
host.name: WKSTN-* AND network.protocol: http AND network.direction: egress AND destination.domain: cdn.golge.xyz
```

Trim the table columns to just `host.name` and the request `query`/URL field. **Finding:** traffic was all **GET requests to 3 distinct `.php` endpoints** — and crucially, **both workstations hit the same 3 endpoints**, implying both machines are infected with the **same malware/C2 implant**, not two separate incidents.

Pivot to `winlogbeat-*` to find the responsible process:
```
host.name: WKSTN-* AND *cdn.golge.xyz*
```

**Finding:** the connection to `cdn.golge.xyz` traces back to a **malicious PowerShell command** — tying this channel back to the same PowerShell-abuse patterns covered in the Execution section (encoded commands, `IEX`, etc.).

---

## Big-Picture Takeaways (for future-me re-reading this cold)

1. **Always pivot index ↔ index.** Network logs (`packetbeat-*`) tell you traffic happened; host logs (`winlogbeat-*`) tell you *what process* did it. Neither alone tells the full story.
2. **Aggregate before you drill.** Don't grep blind — build a Lens table first to find the statistical outlier (most subdomains, least common domain, highest request count), *then* zoom into specifics.
3. **"View surrounding documents" is your time machine.** It's how you find a parent process, or the command issued right before/after a suspicious event.
4. **The same artifacts re-appear across tactics.** `installer.exe` shows up in both Persistence (RunOnceEx) and C2 (Discord). `golge.xyz` / `cdn.golge.xyz` connects DNS tunneling and HTTP C2 to the same campaign. Real intrusions aren't five separate stories — they're one story told across five ATT&CK tactics. Always check if an IOC you already found reappears later.
5. **C2 channel choice tells you about the attacker's evasion priorities:**
   - DNS tunneling → works almost everywhere (DNS is rarely blocked), but high volume/hex subdomains are detectable if you're looking for them.
   - Cloud app abuse (Discord) → blends into "normal" SaaS traffic, hard to block without breaking legitimate use.
   - Custom encrypted HTTP → most flexible/full-featured for the attacker, but a distinctive domain pattern + repeated endpoints is the giveaway.
6. **LOLBAS and scripting abuse exist precisely because they look "boring."** `nslookup.exe`, `certutil.exe`, `chrome.exe` (fake), Python — none of these scream malware on their own. The suspicion always comes from *context*: who's the parent process, what's the destination, what's the command line.

## Gaps / Things to Revisit Next Time

- Why `.lnk` files specifically get weaponized in phishing zips (covered briefly above, but worth a deeper technical pass).
- Anti-forensics timing — *when* in an intrusion attackers typically clear logs, and why.
- Backtracking how `dev.py` (the Python reverse shell) actually landed on disk — the room flags this as a next step but this write-up doesn't trace it.
- Full event trail of everything `installer.exe` did after establishing the Discord C2 channel.
