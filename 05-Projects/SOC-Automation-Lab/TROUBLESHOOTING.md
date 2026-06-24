# TROUBLESHOOTING.md

# SOC Automation Lab Troubleshooting Guide

## Introduction

This document records every major issue encountered while building the SOC Automation Lab.

The purpose is to:

* Help future users reproduce the project

* Document root causes

* Provide validated fixes

* Demonstrate real-world troubleshooting methodology

This document was created during the deployment of:

Windows 11

↓

Sysmon

↓

Wazuh Agent

↓

Wazuh Manager

↓

Shuffle SOAR

↓

VirusTotal

↓

TheHive

---

# Issue #1

## Sysmon Events Visible Locally But Missing in Wazuh

### Problem

Sysmon was successfully installed on the Windows endpoint.

Events appeared in:

Event Viewer

↓

Microsoft

↓

Windows

↓

Sysmon

↓

Operational

However:

Wazuh Dashboard showed little or no Sysmon telemetry.

---

### Symptoms

Observed:

* Sysmon Event ID 1 visible

* Sysmon Event ID 3 visible

* Sysmon Event ID 10 visible

But:

* No corresponding alerts in Wazuh

* Discover contained very few Sysmon events

---

### Investigation

Verified Sysmon locally:

Get-WinEvent `-LogName Microsoft-Windows-Sysmon/Operational`

-MaxEvents 20

Result:

Events successfully returned.

---

### Root Cause

The Wazuh Agent was not subscribed to:

Microsoft-Windows-Sysmon/Operational

The default configuration only monitored:

Application

Security

System

---

### Resolution

Modified:

C:\\Program Files (x86)\\ossec-agent\\ossec.conf

Added:

<localfile>

  <location>Microsoft-Windows-Sysmon/Operational</location>

  <log_format>eventchannel</log_format>

</localfile>

Restarted:

Restart-Service WazuhSvc

---

### Verification

Checked:

ossec.log

Observed:

Analyzing event log:

'Microsoft-Windows-Sysmon/Operational'

Issue resolved.

---

# Issue #2

## Wazuh Discover Showing Only \~24 Events

### Problem

After enabling Sysmon collection, Discover still displayed very few events.

---

### Symptoms

Observed:

Approximately:

24 events

regardless of user activity.

Opening:

* CMD

* PowerShell

* Calculator

* Notepad

did not noticeably increase event volume.

---

### Investigation

Generated test activity:

whoami

ipconfig

net user

Verified:

Events existed in Sysmon.

---

### Root Cause

Discover searches were not showing all collected telemetry.

The issue was visualization rather than ingestion.

---

### Resolution

Enabled raw event archiving.

File:

/var/ossec/etc/ossec.conf

Added:

<logall>yes</logall>

<logall_json>yes</logall_json>

Restarted:

systemctl restart wazuh-manager

---

### Verification

Command:

tail -f /var/ossec/logs/archives/archives.json

Observed:

Sysmon events successfully arriving.

Issue resolved.

---

# Issue #3

## Ping Activity Did Not Generate Expected Network Events

### Problem

Executing:

ping 8.8.8.8

did not immediately generate the expected Sysmon network events.

---

### Investigation

Compared:

whoami

ipconfig

net user

against:

ping

---

### Root Cause

The Olaf Hartong configuration intentionally suppresses large amounts of network noise.

Not all network activity is logged.

---

### Resolution

No action required.

This behavior was expected.

---

### Lesson Learned

High-quality telemetry often means fewer logs, not more logs.

---

# Issue #4

## Wazuh Dashboard Became Extremely Slow

### Problem

Dashboard performance degraded significantly.

Pages loaded slowly.

Discover searches became unreliable.

---

### Investigation

Checked:

free -h

top

df -h

Indexer Health

---

### Findings

CPU:

Idle

Memory:

Available

Disk:

Healthy

Cluster:

Green

---

### Root Cause

Not resource exhaustion.

Dashboard temporarily lost communication with backend components.

---

### Resolution

Validated:

systemctl status wazuh-dashboard

systemctl status wazuh-indexer

systemctl status wazuh-manager

Services healthy.

Issue eventually resolved after dashboard reconnection.

---

# Issue #5

## Wazuh Dashboard Stopped Responding After AWS Restart

### Problem

Dashboard became inaccessible after stopping and starting the EC2 instance.

---

### Symptoms

Browser failed to load.

HTTPS connection failed.

---

### Root Cause

AWS assigned a new public IP address.

---

### Resolution

Updated references using new public IP.

Verified:

https://NEW_PUBLIC_IP

Dashboard accessible again.

---

# Issue #6

## Windows Agent Disconnected After Instance Restart

### Problem

Agent appeared offline.

---

### Root Cause

Agent configuration referenced old public IP.

---

### Resolution

Modified:

ossec.conf

Changed:

<address>OLD_IP</address>

To:

<address>NEW_IP</address>

Restarted agent.

---

### Verification

Agent status:

Active

---

# Issue #7

## Unable to SSH into Wazuh Server

### Problem

SSH connection failed.

---

### Symptoms

Connection timeout.

---

### Investigation

Reviewed:

AWS Security Group

---

### Root Cause

SSH rule no longer matched current public IP.

---

### Resolution

Updated:

TCP 22

Source:

Current Public IP/32

---

### Verification

SSH connection successful.

---

# Issue #8

## Unable to SSH into TheHive Server

### Problem

SSH failed despite running instance.

---

### Root Cause

Same security group restriction issue.

---

### Resolution

Updated SSH rule.

Verified connectivity.

---

# Issue #9

## TheHive Service Failed to Start

### Problem

systemctl status thehive

returned:

failed

---

### Investigation

Validated dependencies.

Checked:

Cassandra

Elasticsearch

TheHive

---

### Findings

Cassandra:

Running

Elasticsearch:

Running

TheHive:

Failed

---

### Resolution

Restarted:

systemctl restart thehive

---

### Verification

Status:

active (running)

---

# Issue #10

## Custom Rule 100002 Not Triggering

### Problem

Mimikatz executed successfully.

No alert generated.

---

### Investigation

Reviewed raw Sysmon event.

Located:

win.eventdata.originalFileName

---

### Root Cause

Rule conditions did not initially align with event fields.

---

### Resolution

Created:

<rule id="100002">

Using:

win.eventdata.originalFileName

Regex:

(?i)mimikatz.exe

---

### Verification

Mimikatz execution generated:

Mimikatz usage detected

---

# Issue #11

## Shuffle Workflow Not Receiving Events

### Problem

Webhook did not trigger.

---

### Investigation

Reviewed:

ossec.conf

integration block

---

### Root Cause

Integration configuration missing or incorrect.

---

### Resolution

Added:

<integration>

<name>shuffle</name>

<hook_url>

WEBHOOK_URL

</hook_url>

<rule_id>100002</rule_id>

<alert_format>json</alert_format>

</integration>

Restarted manager.

---

### Verification

Explore Runs displayed new workflow execution.

---

# Issue #12

## TheHive Authentication Failure

### Error

401 Unauthorized

---

### Root Cause

Authorization header incorrectly configured.

---

### Resolution

Used:

Authorization: Bearer API_KEY

---

### Verification

curl request succeeded.

---

# Issue #13

## TheHive Invalid JSON

### Error

400 Bad Request

Invalid json

JsResultException

---

### Investigation

Compared:

Shuffle request

against

working curl request.

---

### Root Cause

Malformed JSON body generated by Shuffle.

---

### Resolution

Corrected request structure.

Validated against successful curl example.

---

# Issue #14

## sourceRef Validation Failure

### Error

error.expected.jsstring

---

### Root Cause

Numeric value supplied.

Incorrect:

"sourceRef":100002

---

### Resolution

Converted to string.

Correct:

"sourceRef":"100002"

---

### Verification

Validation error disappeared.

---

# Issue #15

## Duplicate Alert Creation Error

### Error

Alert already exists

---

### Message

Alert internal:Wazuh:100002 already exists

---

### Root Cause

TheHive treats:

source + sourceRef

as unique.

Every alert reused:

100002

---

### Resolution

Used unique alert identifiers.

Recommended:

Wazuh Alert ID

---

### Verification

Alert creation succeeded.

---

# Issue #16

## HTTP 404 During API Testing

### Problem

Received:

404 Not Found

---

### Root Cause

Request sent to:

http://SERVER:9000/

instead of:

http://SERVER:9000/api/v1/alert

---

### Resolution

Corrected endpoint path.

---

### Verification

API responded successfully.

---

# Issue #17

## Discover Page Returned "Failed to Fetch"

### Error

TypeError: Failed to fetch

---

### Investigation

Checked:

Dashboard

Indexer

Manager

---

### Root Cause

Temporary communication issue between Dashboard and backend services.

---

### Resolution

Verified service health.

Reloaded dashboard.

Issue resolved.

---

# Final Lessons Learned

1. Always verify telemetry at the source.

2. Validate every integration independently.

3. Use curl before debugging workflow tools.

4. Inspect raw logs before assuming collection failure.

5. Security groups cause many connectivity issues.

6. AWS public IP changes can break multiple components.

7. Detection engineering requires understanding event structure.

8. Alert automation depends heavily on data validation.

9. A successful manual API call is the fastest way to isolate workflow problems.

10. Troubleshooting is a normal part of building production security environments.

