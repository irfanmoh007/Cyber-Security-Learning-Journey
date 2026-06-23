# Fixit — TryHackMe

**Difficulty**: Medium
**Category**: Splunk Configuration / Log Analysis
**Date**: June 2026

---

## Honest Reflection

This room was about fixing a broken Splunk app configuration — 
specifically fixing event boundaries, extracting custom fields, 
and then using those fields to answer investigation questions.

**This is the first room I completed entirely on my own without 
any AI help, Google searches for answers, or writeup references.**

Every question was answered using knowledge I built from the 
Splunk Advanced room combined with hands-on experimentation 
inside the Splunk instance.

**What this room taught me:**
Splunk is only as useful as the configuration behind it. Raw logs 
mean nothing if events are not properly separated and fields are 
not properly extracted. The analyst who configures Splunk correctly 
is the one who can actually investigate effectively.

---

## What This Room Was About

The Fixit app had a broken configuration:
- Logs were being ingested but events were not separated correctly
- Fields like username, department, domain, URI, source IP, 
  and country were not extracted
- Without proper field extraction, searching and investigation 
  was impossible

My job was to fix three configuration files:
- props.conf — defines how Splunk processes log data
- transforms.conf — defines regex field extractions
- fields.conf — defines custom field properties

Then use the fixed configuration to answer investigation questions.

---

## Splunk Configuration Files Explained

### props.conf
Controls how Splunk handles incoming data for a specific sourcetype.
Located at:
/opt/splunk/etc/apps/[appname]/local/props.conf

Key settings used in this room:

**BREAK_ONLY_BEFORE:**
Tells Splunk where one event ends and the next begins.
Used when logs have a consistent pattern at the start of each event.

```ini
[your_sourcetype]
BREAK_ONLY_BEFORE = ^\[Network-log\]:
```

This means: start a new event every time you see `[Network-log]:` 
at the beginning of a line.

**LINE_BREAKER:**
Alternative to BREAK_ONLY_BEFORE. Splits events on a specific 
regex pattern.

**TRANSFORMS:**
Points to field extraction rules defined in transforms.conf

---

### transforms.conf
Defines regex patterns for extracting custom fields from raw events.
Located at:
/opt/splunk/etc/apps/[appname]/local/transforms.conf

Example structure:
```ini
[extract_username]
REGEX = Username:\s(?<username>\w+)
FORMAT = username::$1
```

Fields extracted in this room:
- username
- department
- domain
- URI
- source IP
- country

---

### fields.conf
Defines properties for custom extracted fields.
Located at:
/opt/splunk/etc/apps/[appname]/local/fields.conf

```ini
[username]
INDEXED = true
```

---

## Room Walkthrough

### Finding the App Directory

```bash
find /opt/splunk/etc/apps -name "fixit" -type d
```

Full path: `/opt/splunk/etc/apps/fixit`

**Key knowledge:**
All Splunk apps live under `/opt/splunk/etc/apps/[appname]/`
Standard subdirectory structure:
appname/
├── bin/          ← scripts that generate data
├── default/      ← default configs
├── local/        ← your custom configs go here
└── metadata/

---

### Finding the Network Logs Script

```bash
cat /opt/splunk/etc/apps/fixit/default/inputs.conf
```

inputs.conf defines what data Splunk ingests and from where.
The script path was defined directly inside this file:
`/opt/splunk/etc/apps/fixit/bin/network-logs`

---

### Fixing Event Boundaries

**The problem:**
Splunk was treating multiple log entries as one giant event because 
it did not know where each event started and ended.

**The solution:**
Each log entry started with `[Network-log]:` — a consistent pattern.
Used BREAK_ONLY_BEFORE to tell Splunk this marks the start of 
a new event.

```ini
[network_logs]
BREAK_ONLY_BEFORE = ^\[Network-log\]:
```

**Why BREAK_ONLY_BEFORE over LINE_BREAKER:**
BREAK_ONLY_BEFORE preserves the delimiter as part of the event.
LINE_BREAKER splits on the pattern and can remove it.
For log formats where the header needs to stay in the event, 
BREAK_ONLY_BEFORE is the correct choice.

---

### Extracting Custom Fields

After fixing event boundaries, extracted fields using regex in 
transforms.conf. The key is knowing what the raw log looks like 
first, then writing regex to capture each field value.

**Regex pattern approach:**
fieldname:\s(?<fieldname>[^\s]+)
- `fieldname:` — literal text before the value
- `\s` — whitespace after the colon
- `(?<fieldname>[^\s]+)` — named capture group grabbing the value

---

## Investigation Answers and How I Found Them

**Domain in log data: Cybertees.THM**
After fixing field extraction, ran:
index=* | stats count by domain

**Username field values: 28**
index=* | stats dc(username) as unique_users
dc() = distinct count function in Splunk

**URI field values: 12**
index=* | stats dc(URI) as unique_uris

**Individual /products pages: 2**
index=* URI="/products*" | stats dc(URI)

**URI without file extension: /sales/**
index=* | stats count by URI
Browsed the URI values — /sales/ had no file extension unlike 
others which had .html, .pdf, .php extensions.

**Most active user: Robert Wilson**
index=* | stats count by username | sort -count
Robert Wilson appeared at the top with highest event count.

**Unique IP ranges: 3**
index=* | stats dc(src_ip) by ip_range
Grouped IPs by their first two octets to identify ranges.
Found three distinct ranges in the data.

**User who accessed secret-document.pdf: Sarah Hall**
index=* URI="secret-document.pdf" | stats count by username
Only one username appeared in events matching that URI.

---

## Key Splunk Commands Used

```splunk
# Count events by field value
index=* | stats count by fieldname

# Distinct count of field values
index=* | stats dc(fieldname) as unique_count

# Sort results highest first
index=* | stats count by fieldname | sort -count

# Filter by field value pattern
index=* fieldname="*pattern*"

# Find specific file access
index=* URI="*filename*" | stats count by username
```

---

## Key Lessons Learned

**Event boundary configuration is foundational:**
Before any analysis is possible, events must be correctly separated.
Broken event boundaries make every search result unreliable.

**BREAK_ONLY_BEFORE vs LINE_BREAKER:**
- BREAK_ONLY_BEFORE → keeps the matching line as part of the event
- LINE_BREAKER → splits on the pattern, may remove it

**Field extraction workflow:**
1. Look at raw log format first
2. Identify consistent patterns around each value
3. Write regex with named capture groups
4. Test in Splunk field extractor UI before adding to transforms.conf
5. Verify in fields.conf

**Investigation workflow in Splunk:**
1. Confirm events are correctly separated
2. Verify fields are extracted and populated
3. Use stats count by [field] to understand data distribution
4. Filter progressively to narrow down answers

---

## Splunk Config File Locations Reference
App directory:     /opt/splunk/etc/apps/[appname]/
Input scripts:     /opt/splunk/etc/apps/[appname]/bin/
Default configs:   /opt/splunk/etc/apps/[appname]/default/
Custom configs:    /opt/splunk/etc/apps/[appname]/local/
props.conf:        defines event processing rules
transforms.conf:   defines field extraction regex
fields.conf:       defines custom field properties
inputs.conf:       defines data inputs and sources

---

## MITRE ATT&CK Relevance

Proper Splunk configuration enables detection of:
- T1078 — Valid Accounts (track username field across events)
- T1213 — Data from Information Repositories (document access)
- T1071 — Application Layer Protocol (URI and network analysis)

Without correct field extraction none of these detections work.
Configuration is the foundation of detection capability.

---

## Personal Milestone

This is the first room I completed 100 percent independently.
No AI answers. No writeups. No Google for hints.

The knowledge came from:
- Splunk Advanced TryHackMe room
- Reading Splunk documentation for specific settings
- Trial and error with regex patterns

The methodology that worked:
1. Read the raw log format first
2. Understand what needs to be extracted
3. Write the config
4. Test and verify
5. Then investigate
