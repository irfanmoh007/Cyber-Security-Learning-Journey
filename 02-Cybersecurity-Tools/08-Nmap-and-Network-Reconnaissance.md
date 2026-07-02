# Nmap and Network Reconnaissance

This document outlines the concepts of network reconnaissance (passive and active) and details the implementation and use of **Nmap** (Network Mapper) to discover hosts, scan open ports, identify running services, and execute vulnerability sweeps.

---

## Part 1: Reconnaissance Methodology

Reconnaissance is the initial phase of any penetration test or security assessment (Phase 1 of the Cyber Kill Chain). It is divided into two categories:

### 1. Passive Reconnaissance
Gathering information about a target without directly interacting with their systems. This minimizes the risk of detection.
*   **WHOIS:** Querying databases to find domain ownership, IP ranges, and technical contacts.
*   **DNSDumpster:** Mapping out a target's DNS structure, including subdomains and mail servers.
*   **Shodan:** A search engine for internet-connected devices, showing open ports and banners of a target's public-facing assets.
*   **OSINT (Open Source Intelligence):** Mining public platforms (GitHub, LinkedIn) for leaks, employee details, and technological stacks.

### 2. Active Reconnaissance
Directly interacting with the target systems to map out their network topology, open ports, and system configurations. Active recon carries a high risk of triggering firewall alerts or intrusion detection system (IDS) logs.
*   **Primary Tool:** Nmap.

---

## Part 2: Nmap Scan Types Reference

Nmap determines the state of ports by sending custom packets and analyzing the response.

### 1. TCP SYN Scan (`-sS`)
*   **Mechanism:** Also called a **"half-open"** scan. Nmap sends a `[SYN]` packet. If the port is open, the target replies with `[SYN, ACK]`. Nmap immediately replies with a `[RST]` (reset) to tear down the connection instead of sending the final `[ACK]`.
*   **Advantage:** Fast, stealthy, and does not establish a full TCP connection, meaning the target application often does not log the connection.
*   **Syntax:**
    ```bash
    nmap -sS -Pn 192.168.20.12
    ```
    *(Note: `-Pn` disables ping scans, treating all hosts as online—useful if the target blocks ICMP ping requests).*

### 2. TCP Connect Scan (`-sT`)
*   **Mechanism:** Completes the full TCP 3-way handshake (`SYN` ➔ `SYN,ACK` ➔ `ACK`).
*   **Advantage/Disadvantage:** Does not require root/admin privileges on the scanner machine, but it is highly visible and will be logged by target application servers.
*   **Syntax:**
    ```bash
    nmap -sT 192.168.20.12
    ```

### 3. UDP Scan (`-sU`)
*   **Mechanism:** Sends UDP packets to target ports. If no response is received, Nmap marks the port as "open|filtered." If it receives an ICMP port unreachable error, the port is closed.
*   **Advantage:** Crucial for auditing UDP services (DNS on port 53, SNMP on 161, DHCP on 67).
*   **Disadvantage:** Very slow due to target system rate limits on ICMP responses.
*   **Syntax:**
    ```bash
    nmap -sU 192.168.20.12
    ```

---

## Part 3: Scanning Flags and Service Discovery

To gain a detailed blueprint of a target, analysts combine scanning flags:

*   **`-sV` (Service Version Detection):** Probes open ports to determine the exact software name and version number running on the port.
*   **`-O` (OS Detection):** Uses TCP/IP stack fingerprinting (analyzing how a target responds to custom packet attributes like window size and TTL) to determine the operating system.
*   **`-p-` (All Ports Scan):** Scans all 65,535 TCP ports instead of just the default top 1,000 ports.
*   **`-T0` to `-T5` (Timing Templates):**
    *   `-T0` / `-T1`: Extremely slow, used to evade IDS detection.
    *   `-T4`: Aggressive, optimized for fast scanning on stable local networks.

### Recommended Script for Comprehensive Recon
```bash
nmap -sS -sV -O -p- -T4 -oN scan_results.txt 192.168.20.12
```
*   *`-oN`:* Saves the output in a clean, standard text file format.

---

## Part 4: Nmap Scripting Engine (NSE)

NSE allows users to run scripts to automate tasks like vulnerability detection, advanced service discovery, and brute-forcing.

*   **`-sC` (Default Scripts):** Runs a suite of safe, non-intrusive discovery and vulnerability scripts.
*   **Vulnerability Sweeping:** To check if a service is vulnerable to exploits, run the `vuln` category scripts:
    ```bash
    nmap -Pn --script vuln 192.168.20.12
    ```
*   **Specific Script Execution:**
    ```bash
    # Enumerate SMB shares and check for MS17-010 (EternalBlue)
    nmap -p 445 --script smb-vuln-ms17-010 192.168.20.12
    ```
