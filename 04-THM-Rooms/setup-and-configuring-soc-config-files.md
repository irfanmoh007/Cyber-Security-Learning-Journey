# Splunk Data Ingestion, Parsing, and Configuration Files

## My Notes from TryHackMe SOC Level 2 – Splunk

## Introduction

While learning Splunk through the TryHackMe SOC Level 2 path, I realized that searching logs is only a small part of what Splunk does.

Before analysts can search, visualize, create alerts, or investigate events, Splunk must first:

1. Collect the data
2. Parse the data
3. Extract fields
4. Store the data in indexes
5. Make the data searchable

The entire process is controlled by several configuration files.

Understanding these configuration files helped me understand how Splunk transforms raw logs into searchable security events.

---

# Splunk Data Processing Pipeline

A simplified Splunk data flow looks like this:

```text
Raw Log Source
       │
       ▼
 inputs.conf
       │
       ▼
 props.conf
       │
       ▼
 transforms.conf
       │
       ▼
 fields.conf
       │
       ▼
 indexes.conf
       │
       ▼
 Search Head / Dashboards / Alerts
```

Each configuration file has a specific responsibility.

---

# Why Configuration Files Matter

Imagine a Linux system generates the following log:

```text
May 18 14:22:31 webserver sshd[1254]: Failed password for root from 192.168.1.100
```

To a human, this makes sense.

To Splunk, this is initially just a line of text.

Splunk must determine:

* Where does the event start?
* What is the timestamp?
* What is the hostname?
* What is the source IP?
* Which index should store it?
* Which source type should process it?

Configuration files provide these instructions.

---

# inputs.conf

## Purpose

`inputs.conf` tells Splunk:

* What data to collect
* Where to collect it from
* Which index to store it in
* Which source type to assign

Think of it as the "entry point" of data ingestion.

---

## Example 1: Monitor Linux Syslog

```ini
[monitor:///var/log/syslog]
disabled = false
index = linux_logs
sourcetype = linux_syslog
```

### Explanation

| Setting    | Purpose                      |
| ---------- | ---------------------------- |
| monitor    | Monitor a file               |
| disabled   | Enable or disable collection |
| index      | Destination index            |
| sourcetype | Log format identifier        |

---

## Example 2: Monitor Apache Logs

```ini
[monitor:///var/log/apache2/access.log]
disabled = false
index = web_logs
sourcetype = apache_access
```

Splunk will continuously watch the file and ingest new events.

---

# props.conf

## Purpose

After data is collected, Splunk must understand how to parse it.

`props.conf` controls:

* Timestamp extraction
* Event breaking
* Line merging
* Field extraction rules
* Data formatting

Think of it as the file that teaches Splunk how to read logs.

---

## Example

```ini
[linux_syslog]

TIME_PREFIX = ^
TIME_FORMAT = %b %d %H:%M:%S

SHOULD_LINEMERGE = false

MAX_TIMESTAMP_LOOKAHEAD = 15
```

### Explanation

### TIME_PREFIX

Tells Splunk where the timestamp begins.

```text
May 18 14:22:31
```

---

### TIME_FORMAT

Defines timestamp structure.

```text
%b = Month
%d = Day
%H = Hour
%M = Minute
%S = Second
```

---

### SHOULD_LINEMERGE

```ini
SHOULD_LINEMERGE = false
```

Treat every line as a separate event.

Recommended for most security logs.

---

# Event Breaking Example

Suppose logs arrive like:

```text
ERROR START
User Login Failed
IP: 192.168.1.100

ERROR START
User Login Failed
IP: 192.168.1.101
```

We can tell Splunk where events begin.

```ini
[custom_errors]

SHOULD_LINEMERGE = true
BREAK_ONLY_BEFORE = ERROR START
```

Now each error becomes one event.

---

# transforms.conf

## Purpose

`transforms.conf` performs advanced processing.

Common uses:

* Field extraction
* Data masking
* Routing events
* Dropping unwanted data

Think of it as Splunk's "data manipulation engine."

---

# Example 1: Extract Source IP

Raw event:

```text
Failed login from 192.168.1.100
```

Transform:

```ini
[extract_ip]

REGEX = from\s(\d+\.\d+\.\d+\.\d+)

FORMAT = src_ip::$1

WRITE_META = true
```

Result:

```text
src_ip=192.168.1.100
```

Now analysts can search:

```spl
src_ip=192.168.1.100
```

---

# Example 2: Mask Credit Card Numbers

Raw event:

```text
Credit Card: 4111111111111111
```

Transform:

```ini
[mask_cc]

REGEX = \d{16}

FORMAT = XXXXXXXXXXXXXXXX

DEST_KEY = _raw
```

Result:

```text
Credit Card: XXXXXXXXXXXXXXXX
```

Useful for compliance requirements.

---

# Using props.conf with transforms.conf

The two files often work together.

props.conf:

```ini
[payment_logs]

TRANSFORMS-mask = mask_cc
```

transforms.conf:

```ini
[mask_cc]

REGEX = \d{16}

FORMAT = XXXXXXXXXXXXXXXX

DEST_KEY = _raw
```

When Splunk sees the source type:

```text
payment_logs
```

It automatically applies the transform.

---

# fields.conf

## Purpose

`fields.conf` controls field behavior.

It does not extract fields.

Instead, it defines how fields behave inside Splunk.

---

## Example

```ini
[src_ip]

INDEXED = true

SEARCHABLE = true
```

### Explanation

| Parameter   | Purpose                           |
| ----------- | --------------------------------- |
| INDEXED     | Field available at index time     |
| SEARCHABLE  | Allow searching                   |
| MULTIVALUED | Field can contain multiple values |

---

# indexes.conf

## Purpose

Defines how indexes behave.

Controls:

* Storage location
* Retention period
* Maximum size
* Data lifecycle

Think of it as storage management.

---

## Example

```ini
[linux_logs]

homePath = $SPLUNK_DB/linux_logs/db

coldPath = $SPLUNK_DB/linux_logs/colddb

thawedPath = $SPLUNK_DB/linux_logs/thaweddb

frozenTimePeriodInSecs = 7776000
```

---

## Retention Example

```ini
frozenTimePeriodInSecs = 7776000
```

7776000 seconds

≈ 90 days

After 90 days:

```text
Hot → Warm → Cold → Frozen
```

Old data is removed or archived.

---

# outputs.conf

## Purpose

Used by Universal Forwarders.

Defines where data should be sent.

---

## Example

```ini
[tcpout]

defaultGroup = indexer_group
```

---

```ini
[tcpout:indexer_group]

server = 192.168.1.10:9997
```

### Explanation

The forwarder sends data to:

```text
Indexer IP : 192.168.1.10
Port       : 9997
```

This is one of the most common configurations in Splunk environments.

---

# Creating a Custom Splunk App

During the room, I also learned that custom configurations should generally be stored inside apps.

Create an app:

```text
Settings
 └── Apps
      └── Create App
```

Example:

```text
App Name: SOC Lab
Folder: soc_lab
Version: 1.0
Author: Mohamed Irfan
```

App directory:

```text
/opt/splunk/etc/apps/soc_lab/
```

Inside this app:

```text
default/
local/
metadata/
```

Store custom configuration files in:

```text
local/
```

This prevents updates from overwriting configurations.

---

# Why Use local Instead of default?

Bad practice:

```text
/etc/system/default/
```

Updates may overwrite changes.

Recommended:

```text
/etc/system/local/
```

or

```text
/etc/apps/<app_name>/local/
```

Custom configurations remain safe after upgrades.

---

# Key Takeaways

After completing this room, my biggest realization was that data quality directly impacts investigation quality.

If Splunk cannot properly:

* Identify timestamps
* Break events correctly
* Extract fields
* Assign source types

then searches, dashboards, reports, and alerts become much less useful.

The configuration files may seem intimidating at first, but understanding their role makes the entire Splunk data pipeline much easier to understand.

## Summary

| File            | Purpose                     |
| --------------- | --------------------------- |
| inputs.conf     | Collect data                |
| props.conf      | Parse data                  |
| transforms.conf | Manipulate and extract data |
| fields.conf     | Control field behavior      |
| indexes.conf    | Manage storage              |
| outputs.conf    | Forward data                |

Understanding these files helped me move beyond simply using Splunk and start understanding how Splunk works internally.
