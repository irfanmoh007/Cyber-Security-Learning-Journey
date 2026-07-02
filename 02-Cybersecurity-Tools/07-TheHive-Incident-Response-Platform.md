# TheHive: Incident Response Platform

This document outlines the features and implementation of TheHive as a collaborative incident response and case management platform. It covers incident ticketing, tracking observables, and aligning threat handling workflows with standard incident response lifecycles.

---

## Part 1: What is TheHive?

TheHive is an open-source, scalable Security Incident Response Platform (SIRP) designed to make life easier for SOC analysts. Unlike general IT ticketing systems (such as Jira or ServiceNow), TheHive is built specifically for security investigations. It integrates tightly with SIEMs and SOAR platforms, allowing analysts to:

*   **Correlate Alerts:** Group multiple identical or related alerts into a single investigation case.
*   **Track Observables (IOCs):** Catalog files, IP addresses, domains, and hashes associated with an incident.
*   **Collaborate:** Allow multiple analysts to work on the same case simultaneously, assigning sub-tasks and recording investigation notes in real-time.

---

## Part 2: Incident Response Lifecycle Alignment

The processes managed inside TheHive follow standard Incident Response (IR) frameworks (such as NIST SP 800-61). The platform helps structure and track progress through the four main phases of the IR lifecycle:

```text
 ┌─────────────────┐      ┌─────────────────────────┐
 │ 1. PREPARATION  │─────►│ 2. DETECTION & ANALYSIS │
 └─────────────────┘      └─────────────────────────┘
                                       │
                                       ▼
 ┌─────────────────┐      ┌─────────────────────────┐
 │ 4. POST-INCIDENT│◄─────│ 3. CONTAINMENT,         │
 │    ACTIVITY     │      │ ERADICATION, & RECOVERY │
 └─────────────────┘      └─────────────────────────┘
```

1.  **Preparation:** Hardening hosts, configuring logging (Sysmon, Wazuh), and establishing incident response playbooks and case templates in TheHive before an incident occurs.
2.  **Detection & Analysis:** When an alert is ingested into TheHive, analysts perform validation. They investigate the details, analyze the telemetry, and determine the scope of the compromise.
3.  **Containment, Eradication, & Recovery:** Implementing containment (e.g., isolating a virtual machine or disabling a compromised Active Directory account), eradicating threat elements (deleting malicious scripts, closing ports), and recovering systems (restoring databases, verifying system health).
4.  **Post-Incident Activity (Lessons Learned):** Documenting what went wrong, updating rules to prevent a recurrence, and completing the incident case log in TheHive.

---

## Part 3: Structuring Cases and Observables

### Cases and Alerts
*   **Alerts:** Raw incoming alerts from external systems (like Shuffle or Wazuh). They sit in a triage queue. An analyst can review an alert and either **Promote to Case** or mark it as a **False Positive**.
*   **Cases:** Active investigations. Cases can be created manually or promoted from alerts. They contain tasks and observables.

### Tasks
Tasks are the checklist items that guide an analyst through the investigation (e.g., "Analyze memory dump," "Check firewall logs for outbound traffic," "Verify host isolation").

### Observables
Observables are the technical artifacts associated with the case. When you add an observable to a case, TheHive catalogs it by type:
*   `hash` (MD5/SHA256)
*   `ip` (Source/Destination IP)
*   `domain` (C2 domains)
*   `mail` (Sender email addresses)

---

## Part 4: Automated Ticket Creation in Projects

In our **SOC Automation Lab**, we automated ticket creation using **Shuffle SOAR**. When a critical rule triggered (such as Mimikatz execution), Shuffle executed the following automated workflow to populate TheHive:

1.  **Ingestion:** Shuffle sent an HTTP POST request to TheHive API:
    `POST http://thehive-server:9000/api/alert`
2.  **Payload Details:** The request contained structured incident data:
    ```json
    {
      "title": "Mimikatz Execution Detected on Host WIN11-CLI",
      "description": "Critical process creation event triggered for mimikatz.exe. Threat intelligence has verified this hash.",
      "severity": 3,
      "source": "Wazuh-SIEM",
      "type": "Malware-Alert",
      "sourceRef": "wazuh-alert-991823"
    }
    ```
3.  **Observables Injection:** Shuffle automatically added the threat indicators (the file name, host IP, and the VirusTotal threat reputation score) as **Observables** under the created ticket.
4.  **Result:** Analysts opened TheHive to find a fully populated incident ticket complete with reputation evidence, allowing them to skip triage and immediately begin containment and remediation.
