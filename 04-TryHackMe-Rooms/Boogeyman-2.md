# Boogeyman 2 — TryHackMe

**Difficulty**: Medium
**Category**: DFIR / Memory Forensics
**Date**: 22 May 2026

---

## Honest Reflection

This room introduced me to memory forensics using Volatility 3 for the 
first time in a real investigation context. It was significantly harder 
than Boogeyman 1 because instead of analyzing logs and PCAPs, I was 
digging through a raw memory dump (WKSTN-2961.raw) which is a completely 
different skill.

I completed this room with help from AI for specific technical blockers 
like Volatility plugin errors and tool locations. I did NOT copy answers 
directly — I ran the commands myself, hit real errors, and asked for help 
with those specific errors. That is a step forward from yesterday.

**What I did well today:**
- Found most answers by running Volatility commands myself
- Troubleshot plugin errors independently before asking for help
- Built the attack chain in my head as I went through tasks

**What I still need to improve:**
- Sometimes asked AI to confirm answers instead of trusting myself
- Need to get more comfortable reading raw Volatility output
- Memory forensics is a new skill — need more practice

---

## The Framework — How To Find Answers Without Looking Up Writeups

When stuck on any question, do NOT open Medium or YouTube immediately. 
Work through these five steps first:

**Step 1 — Decode the question verb**
- "downloaded" → something was fetched from outside
- "exfiltrated" → something was sent OUT of the machine
- "executed" → something was run on the machine
- "accessed" → something was opened or read
- "established" → a connection was made

**Step 2 — Match to your available files**
- Memory dump (.raw) → Volatility plugins
- Network traffic → Wireshark / tshark
- Email → Thunderbird or cat
- Logs → jq + grep

**Step 3 — Know what the answer looks like visually**
- PID → a number like 4260 or 6216
- File path → starts with C:\ contains folders and filename
- IP:Port → numbers like 128.199.95.189:8080
- URL → starts with http:// or https://
- Hash → 32 character hex string for MD5

**Step 4 — Pick the right Volatility plugin**
- Finding processes → windows.pstree or windows.pslist
- Finding commands run → windows.cmdline
- Finding files → windows.filescan + grep
- Finding network connections → windows.netscan
- Finding scheduled tasks → strings grep on raw dump

**Step 5 — Use what you already found**
Every answer feeds the next question. PID found in one question 
lets you filter the next command. File path found in one question 
tells you what to grep for next. Always re-read previous answers 
before searching anywhere new.

---

## Attack Chain

1. Attacker sends phishing email from westaylor23@outlook.com
2. Victim maxine.beck receives email with malicious Word document 
   Resume_WesleyTaylor.doc
3. Victim opens document → macro executes → downloads stage 2 payload 
   disguised as update.png from files.boogeymanisback.lol
4. wscript.exe executes the downloaded stage 2 JavaScript payload 
   (update.js saved to C:\ProgramData\update.js)
5. Stage 2 payload downloads malicious binary update.exe from same server
6. Binary renamed to updater.exe and dropped into C:\Windows\Tasks\
7. updater.exe (PID 6216) establishes C2 connection to 128.199.95.189:8080
8. Attacker implants scheduled task for persistence — runs PowerShell 
   with base64 encoded payload from registry every day at 09:00

---
##Questions 1- 3 : You can find the answers just by double clicking the eml file 

Email Analysis

**Difficulty**: Easy
**Files used**: Memory dump + email artifacts

**How to find answers:**

The email artifacts are found inside the memory dump itself. Use 
windows.filescan to locate the cached email attachment:

```bash
vol -f WKSTN-2961.raw windows.filescan | grep -i "Resume_WesleyTaylor"
```

The output shows the full cached path inside the victim's Outlook 
temporary internet files folder.

**Key things to look for:**
- Sender and recipient are visible in the email headers
- Attachment name is visible in filescan output
- MD5 hash requires dumping the file and hashing it

**To get the MD5 hash of the attachment:**
```bash
vol -f WKSTN-2961.raw windows.dumpfiles --virtaddr [address from filescan]
md5sum [dumped file] - this is the correct way of doing this 
but i just downloaded the email attachment and wrote this command md5sum <email-attachment>
```
## Questions 4,5 = you can find the answers by running the downloaded email attachment with olevba like this olevba <downloaded document>
---
## Questions after 5 just read the below hints , you will definitely gets the idea like how to find the answers

Stage 2 Payload Analysis

**Difficulty**: Medium
**Files used**: Memory dump

**How to find answers:**

**Finding the macro download URL:**
The Word document contained a macro that downloaded the stage 2 payload. 
Search memory strings for the attacker's domain:
```bash
strings WKSTN-2961.raw | grep -i "boogeymanisback"
```
The URL containing the download path will appear in the output.

**Finding the process that executed stage 2:**
Use the process tree to see what ran the payload:
```bash
vol -f WKSTN-2961.raw windows.pstree
```
Look for wscript.exe — Windows Script Host is what runs .js and .vbs files.

**Finding the full file path of stage 2:**
Since wscript.exe always takes the script path as an argument, check 
command line arguments:
```bash
vol -f WKSTN-2961.raw windows.cmdline
```
Look for the wscript.exe entry — the path after it is your answer.

**Finding PIDs:**
Same windows.pstree or windows.pslist output gives you both the PID of 
wscript.exe and its parent PID.

**Key pattern learned:**
wscript.exe = Windows Script Host = executes .js and .vbs files
If you see wscript.exe in a process tree during an investigation, 
immediately look at what script it was given as an argument. That 
script is almost always malicious.

---

## C2 Binary Analysis

**Difficulty**: Hard
**Files used**: Memory dump

**How to find the download URL for the malicious binary:**
```bash
strings WKSTN-2961.raw | grep -i "boogeymanisback" | grep -i ".exe"
```
The stage 2 JavaScript payload downloaded a binary — the URL will 
appear in the strings output.

**Finding the C2 process PID and file path:**
```bash
vol -f WKSTN-2961.raw windows.pstree
```
Look for suspicious processes running from unusual locations like 
C:\Windows\Tasks\ — legitimate Windows binaries do not run from Tasks.

**Finding the C2 IP and port:**

Note: windows.netscan and windows.netstat do NOT work reliably in 
Volatility 3 on this VM due to missing symbol tables. This is a known 
limitation. Use strings as a fallback:

```bash
strings WKSTN-2961.raw | grep -E -o "([0-9]{1,3}\.){3}[0-9]{1,3}:[0-9]+"
```

Filter the output to find the external IP connected to the malicious 
PID. Cross-reference with the process you already identified.

**Key pattern learned:**
Malware dropped into C:\Windows\Tasks\ is a red flag. Legitimate 
scheduled tasks exist there but executable binaries living there 
are almost always malicious. Always check the parent process of 
anything suspicious in that folder.

---

##  Persistence Mechanism

**Difficulty**: Very Hard
**Files used**: Memory dump

**Finding the scheduled task command:**

The attacker ran schtasks to create persistence after establishing C2. 
Since this was a short-lived process that had already closed before 
the memory dump was captured, it does NOT appear in windows.cmdline.

Instead grep the raw memory strings:
```bash
strings WKSTN-2961.raw | grep -i "schtasks" | grep -i "create"
```

The full command will appear. It is long — read it carefully.

**Breaking down the command found:**
schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR
'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
-NonI -W hidden -c "IEX ([Text.Encoding]::UNICODE.GetString(
[Convert]::FromBase64String((gp HKCU:\Software\Microsoft
Windows\CurrentVersion debug).debug)))"'

**What each part means:**
- `/Create` → creating a new scheduled task
- `/F` → force overwrite if task already exists
- `/SC DAILY /ST 09:00` → runs every day at 9am
- `/TN Updater` → task is named "Updater" to look legitimate
- `/TR` → what the task actually runs
- `-NonI -W hidden` → PowerShell runs hidden, no window visible
- `IEX` → Invoke-Expression, executes whatever string it receives
- `FromBase64String` → decodes base64 from registry
- `HKCU:\Software\Microsoft\Windows\CurrentVersion debug` → 
  attacker stored the payload inside the registry under a key 
  called "debug" — sneaky because this looks like a legit registry path

**Key pattern learned:**
This is a fileless persistence technique. The actual payload is NOT 
stored as a file on disk. It lives in the registry as a base64 string. 
Every day at 9am PowerShell reads it from registry, decodes it, and 
executes it in memory. This is extremely hard to detect without 
memory forensics because there is no file to find on disk.

---

## Key Volatility 3 Commands Reference

```bash
# Find all running processes in tree format
vol -f WKSTN-2961.raw windows.pstree

# Find command line arguments for all processes  
vol -f WKSTN-2961.raw windows.cmdline

# Search for files in memory
vol -f WKSTN-2961.raw windows.filescan | grep -i "keyword"

# Dump a specific file from memory
vol -f WKSTN-2961.raw windows.dumpfiles --virtaddr [address]

# Find network connections (may not work on all VMs)
vol -f WKSTN-2961.raw windows.netscan

# Search raw memory strings for anything
strings WKSTN-2961.raw | grep -i "keyword"

# Find IP addresses in memory dump
strings WKSTN-2961.raw | grep -E -o "([0-9]{1,3}\.){3}[0-9]{1,3}:[0-9]+"
```

---

## New Techniques Learned Today

- **windows.pstree** shows parent-child process relationships — 
  unusual parent-child pairs indicate malicious activity
- **windows.cmdline** shows arguments passed to processes — 
  this reveals what scripts or files were executed
- **wscript.exe** executing anything is suspicious — it runs 
  VBScript and JavaScript files
- **Fileless malware** stores payload in registry not on disk — 
  only visible through memory forensics
- **strings on raw dump** is the most reliable fallback when 
  Volatility plugins fail
- Processes running from **C:\Windows\Tasks\** with .exe extension 
  are a major red flag

## Where I Struggled Most

- Volatility 3 network plugins (netscan/netstat) failed due to 
  missing symbol tables — had to use strings as fallback
- Scheduled task command was in a terminated process — not visible 
  in windows.cmdline, required raw strings search
- Reading Volatility output is overwhelming at first — grep is 
  essential to filter noise

## What I Will Study Next

- Volatility 3 plugin list and what each one does
- Fileless malware techniques and registry-based persistence
- How to set up and practice memory forensics locally
- Base64 encoded PowerShell payloads — how to decode and read them
