# SIEM Tools: Splunk and Wazuh

## What a SIEM Does

A SIEM collects, stores, searches, correlates, and alerts on security logs. It is one of the main tools used by SOC analysts because it brings evidence from many systems into one place.

Typical data sources include:

- Windows Event Logs
- Sysmon telemetry
- Linux authentication logs
- Firewall logs
- Web server logs
- Cloud logs
- EDR alerts
- IDS/NSM logs from tools like Snort and Zeek

## Splunk

Splunk is a powerful log search and analytics platform. In a SOC lab, I used Splunk to ingest endpoint telemetry and search for attack behavior.

### Splunk Use Cases

| Use Case | Example |
| --- | --- |
| Log search | Find events for a host, user, process, or IP |
| Timeline building | Reconstruct activity before and after an alert |
| Detection queries | Search for suspicious process or network behavior |
| Dashboards | Visualize event counts, hosts, users, and trends |
| Alerting | Trigger alerts when a query matches risky behavior |

### Splunk Analyst Skills

- Understand indexes, sourcetypes, fields, and timestamps
- Write clear SPL searches
- Filter noise without hiding important events
- Build timelines using `_time`
- Extract fields when logs are messy
- Turn repeated searches into alerts or dashboards

Example investigation mindset:

```text
What happened?
Which host did it happen on?
Which user was involved?
What process started it?
What network connection followed?
What changed on disk or in the registry?
```

## Wazuh

Wazuh is an open-source security platform that combines endpoint monitoring, log analysis, file integrity monitoring, vulnerability detection, and alerting. It is very useful for a home SOC lab because it provides an agent, manager, dashboard, and detection-rule workflow.

### Wazuh Use Cases

| Use Case | Example |
| --- | --- |
| Endpoint monitoring | Collect Windows, Linux, and Sysmon logs |
| Rule-based alerting | Trigger custom rules for suspicious activity |
| File integrity monitoring | Watch critical files for changes |
| Vulnerability visibility | Identify weak or outdated packages |
| Compliance-style checks | Monitor configuration and policy issues |

## Splunk vs Wazuh

| Area | Splunk | Wazuh |
| --- | --- | --- |
| Main strength | Flexible search and analytics | Security-focused open-source monitoring |
| Query language | SPL | Wazuh rules, filters, dashboard queries |
| Agent | Universal Forwarder or other inputs | Wazuh Agent |
| Detection style | Search-driven and correlation-driven | Rule-driven and agent-driven |
| Lab value | Excellent for learning SIEM search | Excellent for learning endpoint/SIEM workflow |

## Detection Engineering Workflow

```text
Collect logs
   |
Normalize fields
   |
Search for suspicious behavior
   |
Write a detection rule
   |
Test with lab activity
   |
Tune false positives
   |
Document the rule and response steps
```

## What SIEM Tools Taught Me

SIEM work is not just "searching logs." It requires understanding the system, knowing what normal looks like, and asking better investigative questions. The best detections are specific enough to be useful but broad enough to catch real attacker behavior.
