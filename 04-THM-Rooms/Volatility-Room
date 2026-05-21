My Deep Dive Into Memory Forensics with Volatility 3
Understanding Memmap, HTTP Malware Communication, DLLs, Mutexes, Handles, Malfind, SSDT, Drivers, Modules, DriverScan, and Strings
Author: Mohammad Irfan  
Platform: TryHackMe – Volatility Room  
Topic: Memory Forensics & Malware Analysis
---
Introduction
Today I explored one of the most important areas in Cyber Security and Digital Forensics: Memory Forensics using Volatility 3.
At first, many concepts felt confusing because memory forensics is not like normal file analysis. Instead of looking at files stored on disk, we are investigating the RAM (Random Access Memory) of a computer.
RAM contains:
Running processes
Active malware
Network connections
Injected code
Passwords
HTTP requests
DLLs
Drivers
Hidden malware artifacts
Mutexes
Registry information
Handles
Command history
This documentation contains everything I learned while solving the TryHackMe Volatility room and understanding how malware behaves inside memory.
The goal of this write-up is:
To document my complete learning journey
To make future revision easy
To deeply understand the concepts instead of memorizing commands
To build strong foundations in memory forensics
---
What is Memory Forensics?
Memory Forensics is the process of analyzing the RAM of a computer system to identify:
Running malware
Hidden processes
Injected code
Network activity
Active sessions
Encryption keys
In-memory attacks
Unlike normal disk forensics, memory forensics focuses on what is happening RIGHT NOW inside the operating system.
This is extremely important because modern malware often:
Runs only in memory
Avoids writing files to disk
Injects code into legitimate processes
Hides from antivirus
Uses legitimate Windows tools
This is why RAM analysis is considered one of the most powerful techniques in incident response.
---
Understanding Processes in Memory
Every running application becomes a process.
Examples:
chrome.exe
explorer.exe
notepad.exe
powershell.exe
malware.exe
Each process has:
PID (Process ID)
Memory regions
Threads
DLLs
Handles
Network connections
Permissions
Volatility plugins help us inspect these artifacts.
---
windows.pslist – Listing Running Processes
One of the first plugins used in memory forensics is:
```bash
python3 vol.py -f memory.raw windows.pslist
```
This plugin shows:
Running processes
Their PIDs
Parent processes
Creation time
Exit time
Think of `pslist` as:
> A list of all people currently inside a building.
It tells us WHO exists.
But it does NOT tell us:
What they are doing
What files they opened
Whether they are malicious
What network they connected to
For that, we need deeper plugins.
---
Understanding windows.memmap.Memmap
This was one of the most confusing concepts at first.
But after understanding it using analogies, it became much clearer.
---
What is Memory Mapping?
A process does NOT store all its data in one giant continuous block in RAM.
Instead:
Some memory stores code
Some stores DLLs
Some stores downloaded data
Some stores HTTP requests
Some stores strings
Some stores injected shellcode
All these pieces are scattered across RAM.
The operating system keeps track of these scattered memory regions.
That tracking is called a Memory Map.
---
Storage Locker Analogy for Memmap
Imagine RAM is a giant storage facility with millions of lockers.
A process like `reader_sl.exe` rents many lockers:
Locker #12 → startup code
Locker #90 → DLL data
Locker #405 → downloaded payload
Locker #999 → HTTP request strings
Without memmap:
We only see millions of random lockers.
With memmap:
We get a perfect map showing:
> Which lockers belong to a specific process.
---
windows.memmap.Memmap Command
```bash
python3 vol.py -f Investigation-1.vmem windows.memmap.Memmap --pid 1640
```
This shows:
Virtual memory regions
Memory permissions
Mapped addresses
Memory ownership
---
Using --dump with Memmap
```bash
python3 vol.py -f Investigation-1.vmem -o /home/ubuntu/Desktop/ windows.memmap.Memmap --pid 1640 --dump
```
The `--dump` option extracts ALL memory regions belonging to that process into a dump file.
Example:
```bash
pid.1640.dmp
```
This is extremely useful because:
Instead of searching through an entire 4GB memory dump, we isolate only the suspicious process.
This helps us:
Find malware strings
Find URLs
Find User-Agent values
Find HTTP requests
Extract payloads
Analyze injected code
---
PermissionError Problem and Solution
While using memmap, I encountered:
```bash
PermissionError: [Errno 13] Permission denied
```
Reason:
Volatility tried creating temporary files inside a protected directory.
---
Solution 1 – Using sudo
```bash
sudo python3 vol.py -f Investigation-1.vmem -o /home/ubuntu/Desktop/ windows.memmap.Memmap --pid 1640 --dump
```
---
Solution 2 – Copying the File
```bash
cp /Scenarios/Investigations/Investigation-1.vmem /home/ubuntu/Desktop/
```
Then:
```bash
python3 vol.py -f /home/ubuntu/Desktop/Investigation-1.vmem -o /home/ubuntu/Desktop/ windows.memmap.Memmap --pid 1640 --dump
```
---
Understanding Malware HTTP Communication
One major learning point was understanding:
> Why malware uses HTTP.
---
Why Attackers Use HTTP
Malware communicates with attacker servers using:
HTTP
HTTPS
DNS
TCP
UDP
HTTP is extremely popular because it blends into normal traffic.
Every normal computer constantly sends HTTP requests:
Browsing websites
Updating software
Loading ads
Syncing cloud apps
So when malware sends HTTP traffic:
Firewalls often think:
> “This is normal web browsing.”
This allows malware to bypass many security systems.
---
What is C2 (Command and Control)?
Attackers need communication with infected systems.
This communication is called:
Command and Control (C2).
The malware:
Sends stolen data
Receives commands
Downloads payloads
Updates itself
Reports infection status
HTTP is commonly used for C2.
---
Does Every Process Use HTTP?
No.
Only programs needing internet communication use HTTP.
Examples:
Legitimate network processes:
chrome.exe
firefox.exe
OneDrive.exe
Teams.exe
Processes that usually should NOT use HTTP:
calc.exe
notepad.exe
csrss.exe
fake malware-disguised processes
---
The reader_sl.exe Example
In the investigation:
`reader_sl.exe` was communicating externally.
But:
The REAL Adobe Reader Speed Launcher should NOT normally contact suspicious internet servers.
This became a huge red flag.
Meaning:
The malware disguised itself using a legitimate process name.
---
Identifying HTTP Activity with windows.netscan
Command:
```bash
python3 vol.py -f Investigation-1.vmem windows.netscan
```
This shows:
Active network connections
IP addresses
Ports
Protocols
Associated PIDs
Finding:
PID 1640 connected to port 80.
Port 80 = HTTP.
This strongly indicated HTTP communication.
---
Extracting the HTTP User-Agent
After dumping process memory:
```bash
strings pid.1640.dmp | grep -i "user-agent"
```
This searches memory for HTTP headers.
The User-Agent reveals:
Browser identity
Malware disguise
Communication structure
This is valuable forensic evidence.
---
Understanding the strings Command
This became one of the most important concepts.
---
How strings Works
A memory dump is mostly:
Binary code
Machine instructions
Random bytes
Humans cannot read raw binary easily.
However:
Inside binary data are readable text strings.
Examples:
URLs
Passwords
Usernames
HTTP headers
Malware commands
File paths
The `strings` command scans the binary file and extracts readable text.
---
Simple Explanation of strings
Imagine a huge ocean of random binary garbage.
`strings` acts like a filter.
It ignores garbage and pulls out readable English text.
Example:
Binary:
```text
010101001001001010010
```
Ignored.
Readable text:
```text
Mozilla/5.0
```
Extracted.
---
Why grep is Used with strings
Without grep:
```bash
strings pid.1640.dmp
```
Would output thousands of lines.
Using grep:
```bash
strings pid.1640.dmp | grep -i "user-agent"
```
Filters only lines containing:
```text
user-agent
```
This makes analysis fast and efficient.
---
Understanding DLLs
DLL = Dynamic Link Library.
DLLs are reusable code libraries loaded by programs.
Examples:
Network communication
Graphics
Audio
Keyboard handling
File operations
Instead of every application writing all functionality from scratch, Windows provides DLLs.
---
windows.dlllist
Command:
```bash
python3 vol.py -f Investigation-2.raw windows.dlllist
```
Shows:
DLLs loaded by processes
DLL paths
Memory addresses
---
Problem with Huge DLL Output
Running dlllist globally creates massive output.
Professional workflow:
Always isolate suspicious PID.
Example:
```bash
python3 vol.py -f Investigation-2.raw windows.dlllist --pid 740
```
This reduces noise.
---
Understanding ws2_32.dll
This DLL was critical.
`ws2_32.dll` = Windows Sockets API.
Used for:
Network sockets
Internet communication
TCP/IP operations
When malware loads this DLL:
It often indicates:
Internet activity
Network communication
Socket creation
Possible C2 behavior
---
Faster Hunting with grep
Instead of reading entire output:
```bash
python3 vol.py -f Investigation-2.raw windows.dlllist --pid 740 | grep -i "ws2"
```
This instantly reveals networking DLLs.
---
Understanding Mutexes
Mutex = Mutual Exclusion.
A Mutex prevents multiple instances of malware from running simultaneously.
---
Bathroom Lock Analogy
Imagine one bathroom.
The lock ensures:
Only one person uses it at a time.
That lock acts like a Mutex.
---
Why Malware Uses Mutexes
Malware does NOT want:
Double infection
Re-encryption
Self-corruption
System instability
So malware creates a unique Mutex.
Workflow:
Malware starts
Checks if Mutex already exists
If exists → exit
If not → continue infection
---
Mutexes as Indicators of Compromise (IoCs)
Many malware families use hardcoded Mutex names.
These become:
Indicators of Compromise.
Finding a known Mutex can immediately identify malware.
Example:
```text
Global\MsWinZonesCacheCounterMutexA
```
This is associated with WannaCry.
---
Understanding Handles
Handles were another major concept.
---
What is a Handle?
A Handle is a reference or “ticket” used by Windows to interact with system objects.
Objects include:
Files
Mutexes
Registry keys
Processes
Threads
Events
Sockets
---
Coat Check Analogy for Handles
Imagine giving your coat to a receptionist.
You receive a numbered ticket.
That ticket is the Handle.
Windows uses Handles to track access to objects.
---
Why Mutexes Appear in Handles
A Mutex is a Windows object.
The process holds a Handle pointing to that Mutex.
Therefore:
`windows.handles` can reveal Mutexes.
---
windows.handles Plugin
Command:
```bash
python3 vol.py -f Investigation-2.raw windows.handles --pid 740
```
This reveals:
File handles
Registry handles
Mutex handles
Object types
---
Finding Mutexes with grep
```bash
python3 vol.py -f Investigation-2.raw windows.handles --pid 740 | grep -i "Mutant"
```
Important:
Windows internally calls Mutex objects:
```text
Mutant
```
So filtering for “Mutant” reveals Mutex objects.
---
Example Output
```text
740 @WanaDecryptor@ Mutant Global\MsWinZonesCacheCounterMutexA
```
This identifies the malware.
---
Understanding Malfind
Malfind hunts for injected malicious code.
This plugin became one of the most important concepts.
---
What Does Malfind Detect?
Malfind looks for:
Suspicious memory permissions
Injected executable code
Fileless malware
Hidden shellcode
---
PAGE_EXECUTE_READWRITE
Normal memory permissions are usually separated.
Example:
Read only
Execute only
Write only
But malware often creates memory that is:
Readable
Writable
Executable
At the same time.
This is dangerous.
Malfind detects these suspicious regions.
---
Fileless Malware Detection
Normal programs usually map back to real files on disk.
Injected malware often exists ONLY in RAM.
Malfind detects:
Memory regions with executable code
No corresponding file on disk
This strongly indicates code injection.
---
Airport Security Analogy for Malfind
Malfind acts like airport security.
It scans luggage for hidden compartments.
Even if the outside looks legitimate, it detects hidden malicious content inside memory.
---
Understanding Drivers
Drivers were initially confusing.
A driver is NOT the hardware itself.
The hardware:
Mouse
Keyboard
Printer
USB
These are devices.
The DRIVER is the software translator.
---
Translator Analogy for Drivers
Windows speaks one language.
The hardware speaks another.
The driver acts as the translator.
Without drivers:
Windows cannot communicate with hardware.
---
Why Drivers are Powerful
Drivers run in:
Kernel Space.
Kernel Space has:
Direct hardware access
Full system privileges
Memory control
Device control
This is why malicious drivers are extremely dangerous.
---
Kernel Space vs User Space
User Space:
Chrome
Notepad
Calculator
Restricted.
Kernel Space:
Drivers
Core OS components
God-level privileges.
---
Understanding windows.modules
`windows.modules` shows officially loaded kernel modules/drivers.
Think of it as:
> The official sign-in sheet.
Drivers are supposed to register themselves.
Command:
```bash
python3 vol.py -f memory.raw windows.modules
```
---
Problem with modules
Advanced rootkits can unlink themselves from the official list.
Meaning:
The malware hides from modules.
---
Understanding windows.driverscan
This plugin scans raw memory directly.
Instead of trusting official records, it searches physical RAM structures.
Think of it as:
> Security guards physically walking room-to-room searching for hidden people.
Even if malware hides from official lists:
Its physical memory structures still exist.
driverscan finds them.
---
Difference Between modules and driverscan
Plugin	Purpose
windows.modules	Reads official loaded driver list
windows.driverscan	Scans raw memory for hidden driver structures
---
Understanding SSDT
SSDT = System Service Descriptor Table.
This was one of the most advanced concepts.
---
What is SSDT?
The SSDT acts like:
> A master phone directory for Windows kernel functions.
When programs request system services:
Windows checks the SSDT.
Examples:
Process listing
File operations
Memory access
Registry access
---
How Rootkits Abuse SSDT
Rootkits modify SSDT entries.
Example:
Normal:
```text
Task Manager → Windows Function
```
Rootkit:
```text
Task Manager → Malicious Hook → Fake Results
```
This allows malware to:
Hide processes
Hide files
Hide registry keys
Hide network connections
---
windows.ssdt Plugin
Command:
```bash
python3 vol.py -f memory.raw windows.ssdt
```
This detects suspicious SSDT hooks.
It identifies:
Modified system calls
Redirected functions
Potential rootkits
---
Important Investigation Strategy Learned
One of the biggest lessons:
NEVER analyze entire memory blindly.
Instead:
Find suspicious process
Get PID
Isolate using --pid
Filter output with grep
Analyze focused results
---
Example Investigation Workflow
Step 1 – Find Processes
```bash
windows.pslist
```
Step 2 – Find Network Activity
```bash
windows.netscan
```
Step 3 – Identify Suspicious PID
Example:
```text
PID 1640
```
Step 4 – Dump Process Memory
```bash
windows.memmap.Memmap --pid 1640 --dump
```
Step 5 – Extract Readable Strings
```bash
strings pid.1640.dmp
```
Step 6 – Search for HTTP Artifacts
```bash
grep -i "user-agent"
```
Step 7 – Check DLLs
```bash
windows.dlllist --pid 1640
```
Step 8 – Hunt for Mutexes
```bash
windows.handles --pid 1640 | grep -i "Mutant"
```
Step 9 – Detect Injected Malware
```bash
windows.malfind
```
---
Biggest Lessons Learned
1. Memory Contains Everything
RAM is a goldmine for investigators.
---
2. Malware Loves Legitimate Process Names
Malware disguises itself using names like:
svchost.exe
explorer.exe
reader_sl.exe
---
3. Network Activity is Critical
Unexpected HTTP communication can reveal malware.
---
4. DLLs Reveal Capabilities
Loaded DLLs show:
Networking
Encryption
Registry access
File handling
---
5. Mutexes Act Like Malware Fingerprints
Mutex names can directly identify malware families.
---
6. Handles Reveal What a Process Touches
Handles expose:
Files
Mutexes
Registry keys
Objects
---
7. Malfind is Essential for Injection Detection
It detects:
Hidden shellcode
Fileless malware
Injected memory
---
8. driverscan Beats Hidden Rootkits
Even hidden drivers leave memory structures behind.
---
9. strings is Extremely Powerful
Even raw binary memory often leaks readable artifacts.
---
Final Thoughts
Today’s learning completely changed how I view malware analysis.
I learned that:
Malware rarely operates openly
Attackers abuse legitimate Windows mechanisms
Memory forensics exposes hidden behavior
Volatility is an incredibly powerful framework
Understanding concepts is more important than memorizing commands
The most important thing I realized is:
Cybersecurity investigation is not about blindly guessing answers.
It is about:
Observing clues
Understanding operating system behavior
Connecting concepts logically
Using filtering intelligently
Thinking like an investigator
This journey helped me understand:
Memory structures
Malware communication
Process analysis
DLL investigation
Mutex behavior
Kernel-level analysis
Rootkit detection
String extraction
Process dumping
And most importantly:
It taught me HOW to think during investigations.
---
Commands Reference Cheat Sheet
Process Listing
```bash
python3 vol.py -f memory.raw windows.pslist
```
Network Connections
```bash
python3 vol.py -f memory.raw windows.netscan
```
Memory Map
```bash
python3 vol.py -f memory.raw windows.memmap.Memmap --pid PID
```
Dump Process Memory
```bash
python3 vol.py -f memory.raw -o OUTPUT windows.memmap.Memmap --pid PID --dump
```
DLL Listing
```bash
python3 vol.py -f memory.raw windows.dlllist --pid PID
```
Search DLLs
```bash
grep -i "ws2"
```
Handle Listing
```bash
python3 vol.py -f memory.raw windows.handles --pid PID
```
Find Mutexes
```bash
grep -i "Mutant"
```
Detect Injected Code
```bash
python3 vol.py -f memory.raw windows.malfind
```
Driver Listing
```bash
python3 vol.py -f memory.raw windows.modules
```
Driver Scanning
```bash
python3 vol.py -f memory.raw windows.driverscan
```
SSDT Analysis
```bash
python3 vol.py -f memory.raw windows.ssdt
```
Extract Readable Strings
```bash
strings file.dmp
```
Search User-Agent
```bash
strings file.dmp | grep -i "user-agent"
```
---
Conclusion
This documentation represents my complete understanding and learning from the TryHackMe Volatility room.
The concepts documented here are foundational for:
Malware Analysis
Digital Forensics
Incident Response
Threat Hunting
Memory Analysis
Rootkit Detection
Advanced Windows Internals
This is not just about solving a room.
This is about understanding how attackers operate inside memory and how investigators expose them.
And this is only the beginning of my memory forensics journey.
---
Source notes based on uploaded learning conversation and explanations. fileciteturn0file0
