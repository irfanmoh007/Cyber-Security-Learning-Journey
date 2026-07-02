# CyberChef: The Cyber Swiss Army Knife

This document provides a guide to **CyberChef**, an open-source web application created by GCHQ (Government Communications Headquarters) for analyzing, decoding, and manipulating data. It covers core interface components, common operation categories, and practical recipes for security analysts.

---

## Part 1: What is CyberChef?

CyberChef is designed to enable both technical and non-technical analysts to manipulate data in complex ways without writing custom scripts. It is particularly useful in security operations centers (SOC) for:
*   Decoding obfuscated command-line payloads (e.g., PowerShell Base64).
*   Hashing files or strings to check against threat intelligence.
*   Parsing raw log outputs to extract IP addresses, URLs, and email addresses.
*   Defanging indicators of compromise (IOCs) so they cannot be accidentally clicked in reports.

---

## Part 2: CyberChef Interface Layout

The interface is divided into four main sections:

1.  **Input:** The top-right pane where you paste the raw, obfuscated, or unformatted data you want to analyze.
2.  **Output:** The bottom-right pane where the processed data appears after the recipe runs.
3.  **Operations:** The left column containing a library of hundreds of logical and cryptographic functions (called "operations"), categorized by type (e.g., "Data format," "Encryption / Encoding," "Regex").
4.  **Recipe:** The middle column where you drag and drop operations. Operations run sequentially from top to bottom, feeding their output as the input to the next step.

---

## Part 3: Common Analyst Operations Reference

### 1. Encoding and Decoding
*   **From Base64 / To Base64:** Decodes or encodes binary data to text.
*   **From Hex / To Hex:** Converts hexadecimal byte blocks into readable strings.
*   **URL Decode / URL Encode:** Translates URL-escaped parameters (e.g., converting `%20` back into spaces).

### 2. Cryptography and Hashing
*   **MD5 / SHA-256 / SHA-1:** Generates cryptographic hashes of input text or files. Useful for generating threat signatures.

### 3. Extracting and Parsing
*   **Extract IP addresses:** Scans the input text and extracts all IPv4 and IPv6 addresses.
*   **Extract URLs:** Scans and pulls all web links.
*   **Regular Expression:** Allows analysts to define custom patterns to extract specific parameters from raw logs.

### 4. Defanging (Threat Intel Prep)
*   **Defang IP / Defang URL:** Obfuscates indicators of compromise (e.g., turning `http://malicious-site.com/payload.exe` into `hxxp://malicious-site[.]com/payload[.]exe` and `192.168.1.1` into `192.168.1[.]1`). This prevents analysts or ticketing systems from accidentally clicking them and triggering connections.

---

## Part 4: Practical Recipe Examples

### Recipe 1: Decoding Obfuscated PowerShell Commands
Malware often uses Base64 encoding to hide malicious PowerShell commands from system administrators. Windows logs this as a process command line containing `-EncodedCommand` followed by a long, unreadable string.

*   **Input:** `YQBsAGUAcgB0ACgAMQApADsA`
*   **Recipe:**
    1.  Drag **From Base64** into the Recipe area.
    2.  *(Note: Windows PowerShell encodes strings using UTF-16LE / Unicode, which introduces null bytes. If the output looks like `a.l.e.r.t.(.1.).;.`, you need to remove these).*
    3.  Drag **Decode Text** into the Recipe area, selecting `UTF-16LE (1200)` as the encoding method.
*   **Output:** `alert(1);`

### Recipe 2: Extracting and Defanging IOCs from Raw Logs
If you paste a messy block of firewall log text:

*   **Recipe:**
    1.  Drag **Extract IP addresses** (filters out all non-IP text).
    2.  Drag **Unique** (removes duplicate IPs).
    3.  Drag **Defang IP** (makes them safe for documentation).
*   **Output:** A clean, sorted list of defanged IP addresses (e.g., `10.0.0[.]5`, `192.168.20[.]10`).
