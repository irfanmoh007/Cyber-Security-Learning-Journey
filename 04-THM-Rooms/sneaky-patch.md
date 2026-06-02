# Sneaky Patch — TryHackMe

**Difficulty**: Medium
**Category**: Linux Forensics / Kernel Analysis
**Date**: June 2026

---

## Honest Reflection

This room was about finding a hidden attacker persistence mechanism 
inside a Linux kernel module — something I had zero prior knowledge 
about. I had never investigated kernel modules before this room.

**What I did well:**
- Googled the concept FIRST before touching the room
- Found the suspicious module independently using lsmod
- Noticed the suspicious author and description without being told
- Used strings independently to extract hidden data
- Decoded the flag using CyberChef without help

**What I learned:**
This is the first room where I genuinely followed the correct 
methodology from start to finish. Read the concept, then investigate.

---

## What is a Kernel Module Backdoor?

The Linux kernel can be extended using Loadable Kernel Modules (LKMs).
These are pieces of code that can be inserted into the kernel at 
runtime to add functionality — like device drivers.

Attackers abuse this by:
- Writing a malicious kernel module
- Loading it into the kernel using insmod
- The module runs with the highest possible privileges (ring 0)
- It is extremely hard to detect because it runs below the OS level
- It can hide files, processes, and network connections from the OS

This is called a kernel rootkit — one of the most powerful and 
stealthy persistence mechanisms an attacker can use.

**Why is it dangerous:**
Normal security tools run in userspace. A kernel module runs below 
userspace. The malicious module can intercept and modify what those 
security tools see — effectively making itself invisible.

---

## Investigation Methodology

### Step 1 — Read the room description first
Room said: kernel backdoor, NFS server, persistence.
Question I asked myself: Do I know how Linux kernel modules work?
Answer: No.
Action taken: Googled "Linux kernel modules explained" and 
"how attackers persist via kernel modules" before touching anything.

This 15 minute read gave me enough context to know what to look for.

---

### Step 2 — List all loaded kernel modules

```bash
lsmod
```

This command lists every module currently loaded in the kernel.
Output shows three columns:
- Module name
- Size
- Used by (what else depends on it)

**What to look for:**
Legitimate modules have recognizable names like bluetooth, ext4, 
nvidia. Anything that looks custom, short, or out of place is 
suspicious.

**What I found:**
A module called `spatch` stood out immediately — short, unusual 
name that matched the room theme. In a real investigation you 
would flag anything unfamiliar.

---

### Step 3 — Investigate the suspicious module

```bash
modinfo spatch
```

This shows detailed metadata about the module:
- filename — full path to the .ko file on disk
- description — what the module claims to do
- author — who wrote it
- license — what license it uses

**What legitimate modules look like:**
- description: Driver for XYZ hardware
- author: Linux Kernel Developer
- license: GPL

**What spatch showed:**
- description: cipher
- author: cipher

A one word description and author is highly suspicious. Real kernel 
modules have proper descriptions. This confirmed spatch was malicious.

---

### Step 4 — Extract strings from the module file

```bash
strings [full file path from modinfo output]
```

The strings command extracts all readable ASCII text from a binary 
file. Kernel modules are compiled binaries — you cannot read them 
directly. strings bypasses this by pulling out any text sequences.

**What to look for in strings output:**
- Encoded strings that look like base64 or hex
- Suspicious function names
- Hidden messages or flags
- IP addresses or domain names
- Command strings

**What I found:**
An encoded string near the top of the strings output that looked 
out of place compared to the rest of the technical kernel text. 
It stood out because it did not look like normal kernel code text.

---

### Step 5 — Decode the encoded string

Took the encoded string to CyberChef:
gchq.github.io/CyberChef

Used "From Base64" operation.
Output revealed the flag.

**Key learning:**
Whenever you see a string that looks like random characters — 
especially if it ends in = or == — try base64 decode first in 
CyberChef. That is the most common encoding used to hide data 
inside binaries.

---

## Full Command Sequence

```bash
# Step 1: List all loaded kernel modules
lsmod

# Step 2: Investigate suspicious module
modinfo spatch

# Step 3: Extract readable strings from module binary
strings /path/to/spatch.ko

# Step 4: Copy encoded string to CyberChef and decode
# gchq.github.io/CyberChef → From Base64
```

---

## Key Concepts Learned

**lsmod:**
Lists all currently loaded Linux kernel modules. First tool to 
run when investigating potential kernel-level persistence.

**modinfo [module name]:**
Shows metadata for a specific kernel module. Author and description 
fields are the fastest way to spot malicious modules — legitimate 
ones have proper descriptive text.

**strings [file path]:**
Extracts readable text from any binary file. Essential for 
investigating compiled files like kernel modules, executables, 
and DLLs where you cannot read the source code directly.

**Kernel rootkit persistence:**
Attacker loads malicious .ko file into kernel using insmod. 
Module runs at ring 0 — highest privilege level. Can hide 
processes, files, and network connections from userspace tools.

**File locations to check for malicious modules:**
```bash
# List loaded modules
lsmod

# Find .ko files on disk
find / -name "*.ko" 2>/dev/null

# Check kernel ring buffer for module load messages
dmesg | grep -i "module\|insmod\|loaded"
```

---

## Red Flags That Indicate Malicious Kernel Module

- Single word or meaningless description in modinfo
- Author name that matches attacker theme or is generic
- Module name that does not match any known hardware or feature
- Module loaded from unusual path (not /lib/modules/)
- Encoded strings visible in strings output

---

## What I Did Differently This Room

In previous rooms I jumped straight into tools without understanding 
the underlying concept. This room I:

1. Read the room description
2. Identified the core technology (kernel modules)
3. Asked myself honestly — do I understand this?
4. Answered no — so Googled it first
5. Then started investigating with context already in my head

That one change made everything click. I found the flag 
independently without needing any writeups or AI answers.

---

## What I Will Study Next

- Linux persistence mechanisms beyond kernel modules
- How to detect rootkits using rkhunter and chkrootkit
- MITRE ATT&CK T1547.006 — Kernel Modules and Extensions
- How to analyze .ko files more deeply using objdump
