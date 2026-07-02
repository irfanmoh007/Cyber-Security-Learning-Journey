# Shuffle SOAR and Incident Automation

This document outlines the principles of Security Orchestration, Automation, and Response (SOAR) and details the implementation of automated incident response workflows using Shuffle SOAR. It covers webhook ingestion, indicator enrichment, and API integrations for automated threat handling.

---

## Part 1: What is SOAR?

SOAR stands for **Security Orchestration, Automation, and Response**. It is a collection of software solutions and tools that allow organizations to:

1.  **Orchestrate:** Define how different security tools (SIEM, EDR, Firewall, Email) should work together.
2.  **Automate:** Execute tasks (like threat enrichment, IP blocking, or ticket creation) without human intervention using playbooks/workflows.
3.  **Respond:** Provide a centralized hub for analysts to manage, investigate, and close security incidents.

### Operational Impact
By automating repetitive tier-1 analyst tasks, SOAR dramatically reduces:
*   **MTTD (Mean Time to Detect):** The time it takes to recognize a threat.
*   **MTTR (Mean Time to Respond):** The time it takes to neutralize a threat.
*   **Alert Fatigue:** Preventing analysts from being overwhelmed by thousands of low-priority alerts.

---

## Part 2: Core Shuffle Concepts

Shuffle is an open-source, user-friendly SOAR platform that uses a node-based visual workflow builder.

*   **Workflows:** The complete automation script/playbook.
*   **Triggers (Webhooks):** The entry point. A webhook listens for incoming HTTP POST requests containing JSON alert data sent by security tools (like Wazuh).
*   **Apps:** Connectors for third-party services (e.g., Active Directory, VirusTotal, TheHive, Outlook). Each App has pre-configured actions.
*   **Nodes:** Individual steps inside a workflow (e.g., "Get Hash Info from VirusTotal").
*   **Variables:** Used to pass data between nodes. In Shuffle, this is represented by referencing a previous node's output: `$node_name.field_name`.

---

## Part 3: Automated Threat Handling Workflow Case Study

In our **SOC Automation Lab**, we built an end-to-end automated containment pipeline. The workflow follows these steps:

```text
  Wazuh Alert (Rule 100002)
            │
            ▼
   [Shuffle Webhook Node]
            │
            ▼
    [Regex / JSON Node]  ──► (Extract SHA256 Hash of Mimikatz)
            │
            ▼
   [VirusTotal App Node] ──► (Query hash for reputation score)
            │
            ▼
     [Decision Node]     ──► (Is detection count > 0?)
      /           \
    YES            NO
    /               \
   ▼                 ▼
[TheHive App]   [Log & Drop Alert]
(Create Ticket)
   │
   ▼
[Email App]
(Notify Analyst)
```

### 1. Ingesting Alerts via Webhook
The workflow starts when the Wazuh Manager triggers an alert. Wazuh executes an integration script that sends the JSON alert body to the Shuffle Webhook:
`POST https://shuffle-instance/api/v1/hooks/webhook-id`

### 2. Extracting Indicators of Compromise (IOCs)
We parse the incoming JSON from the Webhook. To extract the file hash, we reference the JSON key path:
`$Wazuh_Webhook.win.eventdata.hashes` or use a regex parser to isolate the SHA-256 value.

### 3. Threat Enrichment via VirusTotal API
Shuffle sends the extracted hash to the VirusTotal App using a pre-configured API key:
*   **Action:** `Get Hash Reputation`
*   **Argument:** `$SHA256_Hash`

### 4. Logic/Decision Branching
We evaluate the results returned by VirusTotal. In Shuffle, we use conditional statements:
*   **Condition:** If `$VirusTotal.positives > 0`
*   **If True (Malicious):** Route the workflow to **TheHive** node to automatically open an incident ticket, and then to the **Email** node to alert the on-call analyst.
*   **If False (Benign):** Stop the workflow and archive the alert.
