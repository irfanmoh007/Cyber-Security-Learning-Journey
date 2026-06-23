# Boogeyman 1 — TryHackMe

**Difficulty**: Medium (but significantly harder in practice)
**Category**: DFIR / SOC
**Date**: 21 May 2026

## Honest Reflection
This room was particularly hard for me to solve. It took me an entire day 
to work through the tasks. I got frustrated multiple times and wanted to 
give up. I am documenting this honestly because the struggle is part of 
the learning process and in this write-up you can find Hints and how i find
the answers for the questions by following this you can also find answers.

**Important note to anyone reading this:**
Do NOT go to Medium pages for direct answers. Instead, when you are stuck, 
search for the TECHNIQUE not the answer. For example: "how to filter HTTP 
POST in Wireshark" or "how to decode hex in bash" — not "Boogeyman 1 
answers." The framework that helped me: identify what TYPE of answer the 
question needs, which FILE it lives in, and what it LOOKS LIKE visually. 
That alone will get you further than any writeup.

## Attack Chain and 
1. Attacker sends phishing email via third-party mail relay
2. Victim receives encrypted zip file — password is included in the email
3. Zip extracted → LNK file found inside
4. LNK file parsed with lnkparse → encoded payload found in command line 
   arguments
5. Payload decoded with CyberChef → reveals download command and C2 domain
6. C2 beacon established → PowerShell polls attacker server every 0.8 
   seconds for commands
7. Attacker downloads enumeration tool from payload delivery server
8. Enumeration tool queries Sticky Notes database → finds KeePass master 
   password stored in a note
9. KeePass database converted to hex and exfiltrated via POST request

---

## Task 1 — VM Setup
Nothing to solve. Just start the machine.

---

## Task 2 — Email Analysis
**Difficulty**: Manageable

This task is straightforward if you follow the investigation guide 
provided in the task itself step by step. Open the dump.eml file in 
Thunderbird and read carefully. All the answers for this task are visible 
from the email headers and attachment.

Key things to look for:
- Sender and recipient addresses are in the email headers
- DKIM-Signature and List-Unsubscribe headers reveal the mail relay service for this user text editor or view souce to find those answers.
- The attachment is an encrypted zip — the password is somewhere in the 
  email body
- Extract the zip, parse the LNK file using lnkparse tool
- The encoded payload is in the Command Line Arguments field of the lnkparse 
  output
- Decode that payload using CyberChef → From Base64 → read what it does , and you can also view the answer for one of the question in the task 3

From task 2 alone you can already start building the attack chain in your 
head. The decoded payload will reveal the first C2 domain.

---

## Task 3 — PowerShell Log Analysis
**Difficulty**: Hard

This is where it gets difficult. Your primary tool here is the 
powershell.json file combined with jq.

The base command pattern to search logs:
```bash
cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[] | 
{ScriptBlockText}' | grep -i "keyword"
```

How to find the domains:
- First domain comes from decoding the payload found in Task 2
- Second domain is found by grepping ScriptBlockText in powershell.json
- Look for script blocks that contain URL patterns

This task took me a long time. The logs are large and the answers are 
buried. Be patient. Change your grep keyword if one does not work. Try 
terms like the domain parts you already know, or common PowerShell 
download commands like "iwr", "invoke-webrequest", "downloadstring".
you can find all the answers to this task by only using the correct grep keyword .

## Task 4 — Exfiltration Analysis
**Difficulty**: Very Hard (hardest task in the room)

**Finding the enumeration tool:**
The payload delivery server (files.bpakcaging.xyz) is different from the 
C2 server. This server delivers tools to the victim machine. Search the 
PowerShell logs for download commands pointing to this server. The tool 
name is in the URL. Once you have the tool name, check what web server 
software is running on that delivery server — the answer is visible in the 
HTTP response headers in Wireshark when you filter traffic to that domain.

**Finding what file was accessed:**
Search PowerShell logs for the enumeration tool name. The command it ran 
will show the exact file path it accessed.

**Finding the software:**
Look at the file path from the previous answer. The folder name contains 
the answer directly.

**Finding the exfiltrated file:**
Run this against the pcap:
```bash
strings capture.pcapng | grep -i ".zip\|.txt\|.db\|.xlsx\|.sqlite\|.kdbx"
```

**Finding the encoding used:**
Search PowerShell logs for the exfiltrated filename. The script that 
handles it will show the encoding method clearly.

**Finding the exfiltration tool:**
Same script block as above. One word answer. Read the sending mechanism.

**Finding the password of the exfiltrated file:**
This is tricky. The attacker queried the Sticky Notes database to find it. 
The output was sent back to C2 via POST request. Decode the POST body:
```bash
tshark -r capture.pcapng -Y "http.request.method==POST && tcp.port==8080" \
-T fields -e http.file_data 2>/dev/null | tr ' ' '\n' | \
awk '{printf "%c", $1}' | strings | grep -A 3 "Master Password"
```
The sticky note content after "Master Password" is the answer.

**Finding the credit card number:**
I will be completely honest — I could not find this answer on my own after 
spending a long time trying. I looked up a walkthrough for this final 
question. It requires reconstructing the KeePass database file from the 
PCAP and opening it with the master password found above. This is 
intermediate-level forensics and beyond what I had learned at this point. 
I will come back to this technique after learning more about file 
reconstruction from packet captures.

---

## Key Techniques Learned
- Encoded strings in HTTP URIs end with = or == and look like random 
  characters
- PowerShell logs record every command run — grep them with specific 
  keywords not generic ones
- Sticky Notes stores data in plum.sqlite — attackers use this for 
  credential hunting
- Hex exfiltration: file bytes converted using ToString X2 then sent as 
  space-separated decimals via POST
- When POST body looks like numbers — convert with tr and awk to readable 
  text

## Where I Struggled Most
- Did not recognise encoded strings visually until explained
- Multiple times the answer was already in my terminal output but I missed 
  it by not reading carefully enough
- File reconstruction from PCAP — this is a gap I still need to close

## What I Will Study Next
- CyberChef encoding/decoding practice (base64, hex)
- Common PowerShell attack commands and what they do
- File reconstruction from PCAP captures


## The Framework — How To Find Answers Without Looking Up Writeups

When you are stuck on any question, do NOT open Medium or YouTube 
immediately. Instead, work through these five steps first:

**Step 1 — Decode the question verb**
Pull out the key action word and map it:
- "downloaded" → something was fetched from outside
- "exfiltrated" → something was sent OUT of the machine
- "executed" → something was run on the machine
- "accessed" → something was opened or read
- "encoded" → something was transformed to look like gibberish

**Step 2 — Match to your available files**
- PowerShell activity → powershell.json
- Network traffic in or out → capture.pcap or capture.pcapng
- Email details → dump.eml
- Always ask: which file would record this type of activity?

**Step 3 — Know what the answer looks like visually**
- Encoded string → random characters, often ends in = or ==, 
  sits after ? in a URL or inside a POST body
- File name → has an extension like .exe .db .zip .kdbx
- IP address → four numbers separated by dots
- Command → readable text starting with powershell/cmd keywords
- Domain → words separated by dots like evil.attacker.com
- Parameter → sits after ? in a URL like ?q=something

**Step 4 — Pick the right tool**
- pcap or pcapng → Wireshark filters or tshark commands
- json logs → jq combined with grep
- eml file → Thunderbird or just cat it and read

**Step 5 — Use what you already found**
Half the answers are hidden inside previous answers. Always re-read 
what you already have before searching anywhere else. File paths 
contain software names. URLs contain tool names. Script blocks 
contain file names. One answer almost always leads to the next.

---

**When you are genuinely stuck after trying all five steps:**
Search for the TECHNIQUE, not the answer.
- Good: "how to decode hex string in bash"
- Good: "how to filter POST requests in Wireshark"
- Bad: "Boogeyman 1 TryHackMe answers"
- Bad: "Boogeyman 1 walkthrough"

The difference between searching for technique vs answer is the 
difference between actually learning and just collecting flags.
