# Boogeyman 3 — TryHackMe

**Difficulty**: Hard
**Category**: DFIR / Threat Hunting / Lateral Movement

---

## Honest Reflection

This was not the hardest Boogeyman room  but it is quite hard . It covered a full attack 
chain from initial access all the way to ransomware deployment, 
including lateral movement across multiple machines and a DCSync 
attack against a domain controller. This is real enterprise-level 
incident response territory.

**What I did well:**
- Found most early-stage answers independently using Kibana search
- Built logical search queries based on known filenames and indicators
- Connected answers from one question to feed the next investigation
- Switched machine context (workstation to DC) at the right time

**What I need to improve:**
- UAC bypass techniques — had no idea what fodhelper.exe was, needed AI
- Lateral movement forensics — kept finding powershell.exe instead of 
  the correct parent process, needed Google
- File share enumeration events — did not know which Event IDs to look 
  for, needed Google
- Need to slow down and investigate properly instead of stumbling onto 
  answers by browsing events randomly

**Honest admission:**
Two answers were found by Googling directly. I am documenting this 
honestly because knowing where my gaps are is more valuable than 
pretending I found everything myself.

---

## The Framework — Finding Answers in Kibana/ELK

**Step 1 — Start with what you know**
Every investigation starts with a known indicator. In this room that 
was the original phishing attachment filename. Type that into the 
Kibana search bar first. Everything else branches from there.

**Step 2 — Follow the chain forward**
Each answer gives you a new indicator to search for next:
- Found a filename → search that filename
- Found a PID → filter by that PID
- Found a username → filter by that username
- Found an IP → filter by that IP

**Step 3 — Add useful columns**
In Kibana, add columns for:
- process.command_line
- destination.ip and destination.port
- user.name
- host.name
- event.id

These four columns answer most questions without deep digging.

**Step 4 — Switch host context when needed**
When the investigation moves to a second machine, update your 
host.name filter. Don't keep searching the wrong machine.

**Step 5 — Know your Event IDs**
- Sysmon Event ID 1 → Process creation (commands run)
- Sysmon Event ID 3 → Network connection
- Sysmon Event ID 11 → File created
- Sysmon Event ID 13 → Registry value set
- Windows Event ID 5140 → Network share accessed
- Windows Event ID 5145 → File inside share accessed
- Windows Event ID 4624 → Successful logon

---

## Full Attack Chain

1. Victim receives phishing email with malicious attachment 
   ProjectFinancialSummary
2. Victim opens attachment → stage 1 payload executes (PID 6392)
3. Stage 1 uses xcopy.exe to copy review.dat from D:\ to Temp folder
4. rundll32.exe executes review.dat as a DLL → C2 connection established
5. Scheduled task named "Review" created for persistence
6. C2 connection made to 165.232.170.151:80
7. Attacker discovers local admin access
8. UAC bypass via fodhelper.exe registry manipulation
9. Attacker downloads mimikatz from GitHub to dump credentials
10. Credentials dumped → itadmin:F84769D250EB95EB2D7D8B4A1C5613F2 harvested
11. Attacker uses itadmin to enumerate network shares
12. IT_Automation.ps1 accessed from remote share — contains credentials
13. New credentials found: QUICKLOGISTICS\allan.smith:Tr!ckyP@ssw0rd987
14. Lateral movement to WKSTN-1327 using those credentials
15. wsmprovhost.exe spawns malicious command on second machine
16. Second machine credentials dumped: administrator:00f80f2538dcb54e7adc715c0e7091ec
17. Attacker reaches domain controller — DCSync attack performed
18. backupda account hash dumped from DC
19. Ransomware binary downloaded from http://ff.sillytechninja.io/ransomboogey.exe

---

## Task by Task Breakdown

### Initial Compromise

**Q: PID of process that executed stage 1 payload**

How I found it:
Searched the phishing attachment filename in Kibana search bar. 
The first process creation event showed the PID directly.
search: "ProjectFinancialSummary"
look at: process.pid field

**Key pattern:** Always start with the filename you already know. 
It anchors the entire investigation.

---

**Q: Full command-line of file implant execution**

How I found it:
Same search as above. xcopy.exe appeared in the command_line column 
copying review.dat from D:\ to Temp folder.
search: "ProjectFinancialSummary"
look at: process.command_line column

**Key pattern:** xcopy.exe is a legitimate Windows binary used to 
copy files. Attackers abuse it to implant payloads to writable 
locations like AppData\Temp.

---

**Q: Full command-line of implanted file execution**

How I found it:
Searched "review.dat" in Kibana. Found rundll32.exe executing it 
with DllRegisterServer as the entry point.

**Key pattern:** rundll32.exe executing a .dat file with 
DllRegisterServer is a massive red flag. Legitimate DLLs have 
.dll extension. .dat is used to disguise malicious DLLs.

---

**Q: Name of scheduled task created for persistence**

How I found it:
Still within the review.dat search results. Scheduled task creation 
event showed task name "Review" — named to look legitimate.

**Key pattern:** Attackers name scheduled tasks to blend in with 
legitimate Windows tasks. Generic names like "Review", "Update", 
"Updater" are red flags when created by unexpected processes.

---

### C2 Connection

**Q: IP and port of C2 connection**

How I found it:
Added destination.ip and destination.port as columns in Kibana. 
Filtered events from the review.dat process. External IP appeared 
clearly.
add columns: destination.ip, destination.port
filter by: the PID or filename from previous answers

---

### Privilege Escalation

**Q: Process used for UAC bypass**

How I found it:
Did not find this independently. Asked AI for help.

**What I learned — fodhelper.exe UAC bypass:**
- fodhelper.exe is a legitimate Windows binary that auto-elevates 
  without a UAC prompt
- Attackers modify registry key:
  HKCU:\Software\Classes\ms-settings\Shell\Open\command
- They point that key to their malicious payload
- When fodhelper.exe runs, it reads that registry key and executes 
  the attacker's command with admin privileges
- The victim sees no UAC prompt because Windows trusts fodhelper.exe

**How to detect it in logs:**
- Sysmon Event ID 13 → registry modification to ms-settings path
- Sysmon Event ID 1 → fodhelper.exe spawning unexpected child process

**Key pattern:** Any trusted Windows binary spawning an unexpected 
child process like cmd.exe, powershell.exe, or a script is suspicious.

---

**Q: GitHub link used to download credential dumping tool**

How I found it:
Searched "github" in Kibana search bar. The download event appeared 
immediately showing the full mimikatz URL.
search: "github"
look at: process.command_line or url fields

**Key pattern:** Attackers frequently pull tools from GitHub during 
post-exploitation. Searching "github" in process command lines is a 
quick way to find tool downloads.

---

### Credential Dumping and Lateral Movement

**Q: Username and hash of dumped credentials**

How I found it:
Filtered by user.name field in Kibana left panel. Found itadmin 
as a username with multiple events. Credential dump output showed 
the NTLM hash.

**What is an NTLM hash:**
Windows stores passwords as NTLM hashes instead of plaintext. 
Mimikatz extracts these from memory. Attackers can use the hash 
directly without cracking it — this is called Pass-the-Hash.

---

**Q: File accessed from remote share**

How I found it:
Did not find this independently. Googled the answer.

**What I should have done:**
I should have done research on remote share and find the answer 
but i did the complete opposite.

---

**Q: New credentials discovered from remote file**

How I found it:
Browsed events and spotted it. Not a clean methodology.

**What I should have done:**
The file IT_Automation.ps1 is a PowerShell automation script. 
These scripts frequently contain hardcoded credentials for running 
automated tasks. Should have looked at the file content events 
or the command output returned after reading the file.

---

**Q: Hostname of lateral movement target**

How I found it:
Found it in the same event line as the credentials. The destination 
hostname was visible in the connection event.

---

**Q: Parent process of malicious command on second machine**

How I found it:
Searched Google because I kept finding powershell.exe.

**What I learned — wsmprovhost.exe:**
- wsmprovhost.exe = Windows Remote Management Provider Host
- It is the legitimate process that handles WinRM remote commands
- When an attacker uses Enter-PSSession or Invoke-Command to run 
  commands remotely, wsmprovhost.exe is the parent process on the 
  receiving machine
- Seeing wsmprovhost.exe spawn cmd.exe or powershell.exe on a 
  machine means someone ran remote commands against it

**How to find it next time:**
Switch host.name filter to the target machine (WKSTN-1327).
Look for process creation events where parent process is unusual.
wsmprovhost.exe as parent = remote command execution via WinRM.

---

**Q: Credentials dumped from second machine**

How I found it:
Filtered user.name on the second machine. Found administrator 
account hash in the mimikatz output events.

---

### Domain Controller Compromise

**Q: Account dumped via DCSync aside from administrator**

How I found it:
Switched host.name filter to the domain controller (DC01). 
DCSync attack events showed which accounts were targeted. 
Found backupda as the second account.

**What is DCSync:**
DCSync is an attack where the attacker pretends to be a domain 
controller and requests password replication data from the real DC. 
This gives them NTLM hashes for any account in the domain without 
ever logging into the DC directly.

It requires Domain Admin or replication privileges to work.
In logs it appears as Windows Event ID 4662 with specific 
replication permissions requested.

---

**Q: Link used to download ransomware**

How I found it:
Searched "http" in Kibana filtered on the DC. Found the download 
event showing ransomboogey.exe being pulled from the attacker's server.
search: "http" or "ransomware" or ".exe"
filter: host.name = DC

---

## Key Techniques Learned Today

**fodhelper.exe UAC bypass:**
Modifying HKCU:\Software\Classes\ms-settings\Shell\Open\command 
to execute malicious payload with auto-elevated privileges.

**rundll32.exe abused to execute disguised DLL:**
Legitimate Windows binary used to run malicious .dat file as a 
DLL — bypasses some security controls that watch for .dll execution.

**Credential theft flow:**
mimikatz → dump NTLM hashes → Pass-the-Hash → access other machines 
without knowing plaintext password.

**Lateral movement via WinRM:**
Attacker uses stolen credentials with PowerShell remoting. 
wsmprovhost.exe is the telltale parent process on the victim machine.

**DCSync attack:**
Attacker with domain privileges mimics DC replication to steal 
all password hashes from Active Directory without touching the DC.

**IT automation scripts as credential stores:**
PowerShell scripts used for automation frequently contain hardcoded 
credentials — these are high value targets for attackers enumerating 
file shares.

---

## Kibana Investigation Tips Learned

- Start every investigation with a known filename or indicator
- Add process.command_line, destination.ip, user.name, host.name 
  as custom columns immediately
- Filter user.name to track attacker movement after credential theft
- Switch host.name filter when investigation moves to new machine
- Search "github" to find tool downloads
- Search "http" to find payload downloads
- Event ID 5145 = file share access — use this for share enumeration

## Gaps to Close

- Practice identifying UAC bypass techniques in logs
- Learn WinRM lateral movement forensics properly
- Study DCSync attack detection using Event ID 4662
- Practice file share enumeration detection using Event IDs 5140/5145
