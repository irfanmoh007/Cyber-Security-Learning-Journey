# Cybersecurity Frameworks: MITRE ATT&CK & Cyber Kill Chain

This document outlines the core models used to analyze, categorize, and defend against cyber threats. It covers the Lockheed Martin Cyber Kill Chain, the MITRE ATT&CK Framework, Threat Intelligence Platforms (TIPs), and the Pyramid of Pain. It concludes with a case study mapping our practical attack simulations to these models.

---

## Part 1: Lockheed Martin Cyber Kill Chain

The Cyber Kill Chain is a linear model representing the stages of an external cyberattack. Breaking the chain at any stage stops the attack from succeeding.

```text
  1. Recon ──► 2. Weaponize ──► 3. Deliver ──► 4. Exploit ──► 5. Install ──► 6. C2 ──► 7. Action
```

1.  **Reconnaissance:** Gathering target information (e.g., email harvesting, port scanning via Nmap).
2.  **Weaponization:** Pairing an exploit with a backdoor payload into a deliverable file (e.g., creating a malicious `.exe` with `msfvenom`).
3.  **Delivery:** Transmitting the payload to the victim (e.g., via phishing email, drive-by download, or hosting it on a Python web server).
4.  **Exploitation:** Executing the malicious code on the target system (e.g., the user double-clicks the payload).
5.  **Installation:** Establishing a persistent presence on the host (e.g., adding registry run keys or installing malware services).
6.  **Command and Control (C2):** The compromised system opens a communications channel back to the attacker's controller (e.g., a Meterpreter session connecting to a Kali handler on port `4444`).
7.  **Actions on Objectives:** Attacking the final target (e.g., data exfiltration, system destruction, or credential dumping via Mimikatz).

---

## Part 2: MITRE ATT&CK Framework

The MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) framework is a detailed matrix cataloging real-world cyber adversary behaviors. It is divided into:

*   **Tactics:** The attacker's immediate goal (e.g., *Initial Access*, *Persistence*, *Credential Access*). There are 14 Tactics in the Enterprise matrix.
*   **Techniques:** How the attacker achieves a Tactic (e.g., under *Credential Access*, the attacker can use the Technique *OS Credential Dumping*).
*   **Procedures:** The specific tool or software command used (e.g., using `mimikatz` command `sekurlsa::logonpasswords` to extract passwords).

### Mapping Detections
SIEM and EDR platforms map custom detection rules to MITRE ATT&CK IDs (e.g., rule `100002` in Wazuh maps to **T1003 - OS Credential Dumping**) so that analysts immediately understand the attacker's objective.

---

## Part 3: Threat Intelligence and the Pyramid of Pain

Threat Intelligence involves collecting and sharing information about current threats, typically in the form of **Indicators of Compromise (IOCs)**. These are shared via **Threat Intelligence Platforms (TIPs)** like MISP or AlienVault OTX.

### The Pyramid of Pain
Created by David Bianco, this model explains how difficult it is for an attacker to change their campaign indicators when a defender blocks them.

```text
           ▲  
          / \     TTPs (Tough - e.g., changing their complete exploit method)
         /   \  
        /     \   Host/Network Artifacts (Annoying - e.g., registry keys, TCP signatures)
       /       \  
      /         \ Domain Names (Simple - e.g., buying a new C2 domain)
     /           \  
    /             \ IP Addresses (Easy - e.g., spinning up a new VPS node)
   /               \  
  /                 \ Hash Values (Trivial - e.g., appending a null byte to change MD5)
 └───────────────────┘
```

*   **Hash Values (Trivial):** Blocking file hashes is easy for defenders, but trivial for attackers to bypass by changing a single byte in the file.
*   **IP/Domains (Easy/Simple):** Attackers can easily purchase new domains or change IPs.
*   **TTPs (Tough):** If you detect and block the attacker's core behavior (TTPs), you force them to learn an entirely new operational behavior, which is highly effective.

---

## Part 4: Practical Lab Mapping Case Study

Here is how the attack simulation in our **SOC Basic HomeLab** and **SOC Automation Lab** maps to these frameworks:

| Attack Phase | Action Taken in Lab | Cyber Kill Chain Stage | MITRE ATT&CK Tactic / Technique |
| :--- | :--- | :--- | :--- |
| **Reconnaissance** | Scanned victim VM using Nmap | Stage 1: Reconnaissance | **Discovery** <br> (T1046: Network Service Scanning) |
| **Weaponization** | Created payload using `msfvenom` | Stage 2: Weaponization | N/A (Attacker Preparation) |
| **Delivery** | Hosted file on Python HTTP server, downloaded on Windows VM | Stage 3: Delivery | **Execution** <br> (T1204.002: User Execution - Malicious File) |
| **Exploitation** | Executed payload executable | Stage 4: Exploitation | **Execution** <br> (T1204: User Execution) |
| **Command & Control** | Opened Meterpreter connection back to Kali Linux | Stage 6: Command & Control | **Command and Control** <br> (T1090: Proxy / C2 Channel) |
| **Actions on Objectives** | Executed Mimikatz to dump credential hashes | Stage 7: Actions on Objectives | **Credential Access** <br> (T1003.001: LSA Secrets / Credential Dumping) |
