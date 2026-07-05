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

## Part 3: Host Discovery and DNS Resolution Options

Host discovery is the process of mapping out active IP addresses on a network before beginning port scans. By default, Nmap performs host discovery and then port-scans any hosts it finds online.

### 1. Disabling Port Scanning (`-sn`)
If you are only interested in identifying active hosts on a subnet without performing any port scans, use the `-sn` flag.
*   **Syntax:**
    ```bash
    nmap -sn 10.200.6.0/24
    ```
    *Note: Omitting `-sn` defaults to scanning all live hosts for open ports, which generates substantial logs and traffic.*

### 2. Listing Targets Without Scanning (`-sL`)
The list scan flag (`-sL`) compiles and displays a list of all hosts in the target range without sending any packets to them. It is used to verify target ranges.
*   **Stealth Evasion Flag (`-n`):** By default, `-sL` performs reverse DNS lookups on the IPs, generating logs on the target's DNS server. Adding `-n` disables DNS resolution, making this completely stealthy.
*   **Syntax:**
    ```bash
    nmap -sL -n 10.10.12.13/29
    ```

### 3. DNS Lookup Customization
*   **`-n` (No DNS resolution):** Prevents Nmap from performing reverse DNS lookups on active hosts. This speeds up scans and avoids generating DNS query logs on target servers.
*   **`-R` (Always DNS resolve):** Forces Nmap to perform reverse DNS lookups on every single IP in the range, active or inactive.

---

### 4. Advanced Host Discovery Techniques (Ping Scans)

When attempting to discover live hosts, different packet types can be used to bypass firewall filters. 

| Scan Type | Mechanism & Purpose | Command Example |
| :--- | :--- | :--- |
| **ARP Ping** | Uses local ARP requests. Highly reliable on the same local subnet. | `sudo nmap -PR -sn 10.200.6.0/24` |
| **ICMP Echo Ping** | Sends standard ICMP echo requests (ping). Frequently blocked by internet-facing firewalls. | `sudo nmap -PE -sn 10.200.6.0/24` |
| **ICMP Timestamp Ping** | Sends ICMP timestamp requests. Bypasses firewalls that block standard echo requests but permit timestamp queries. | `sudo nmap -PP -sn 10.200.6.0/24` |
| **ICMP Address Mask Ping** | Requests the subnet mask of the target. | `sudo nmap -PM -sn 10.200.6.0/24` |
| **TCP SYN Ping** | Sends an empty `[SYN]` packet to specified ports. If target replies with `[RST]` or `[SYN, ACK]`, the host is alive. | `sudo nmap -PS22,80,443 -sn 10.200.6.0/30` |
| **TCP ACK Ping** | Sends an empty `[ACK]` packet. Since there is no active session, target replies with `[RST]`, revealing it is alive. | `sudo nmap -PA22,80,443 -sn 10.200.6.0/30` |
| **UDP Ping** | Sends UDP packets to specified ports (like port 53). If target returns an ICMP port unreachable error, it is alive. | `sudo nmap -PU53,161,162 -sn 10.200.6.0/30` |

*Remember to append `-sn` if you only want to discover hosts without initiating a port scan on live hosts.*

---

## Part 4: Scanning Flags and Service Discovery

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

## Part 5: Nmap Scripting Engine (NSE)

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
