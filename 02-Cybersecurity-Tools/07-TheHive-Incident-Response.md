# TheHive Incident Response Platform

## What TheHive Is

TheHive is an incident response and case management platform. It helps SOC teams turn alerts into structured cases, assign tasks, track evidence, and document response work.

In a SOC workflow, TheHive is where an alert becomes an investigation.

## Core Concepts

| Concept | Meaning |
| --- | --- |
| Alert | Incoming security signal from SIEM, EDR, SOAR, or another tool |
| Case | Formal investigation created from an alert |
| Observable | Evidence item such as IP, domain, hash, URL, email, or username |
| Task | Analyst action item |
| Severity | Risk level of the case |
| TLP/PAP | Sharing and handling labels |

## Use Cases

- Convert SIEM alerts into investigation cases
- Track analyst tasks during incident response
- Store observables from an alert
- Document findings and decisions
- Coordinate handoff between analysts
- Preserve investigation history for reporting

## Example SOC Workflow

```text
Wazuh alert triggers
      |
      v
Shuffle receives webhook
      |
      v
Hash or IP is enriched
      |
      v
TheHive alert is created
      |
      v
Analyst opens a case
      |
      v
Tasks, evidence, and response notes are tracked
```

## Good Case Notes Should Include

- Alert name and source
- Affected host and user
- Timestamp range
- Triggering detection rule
- Observables found
- Timeline of activity
- Analyst conclusion
- Containment or remediation action
- Whether the alert was true positive, false positive, or benign true positive

## Why TheHive Matters

Without case management, SOC work can become scattered across chats, screenshots, tickets, and memory. TheHive gives the investigation a single place to live.

For my learning path, TheHive connects technical detection work with real incident response discipline.
