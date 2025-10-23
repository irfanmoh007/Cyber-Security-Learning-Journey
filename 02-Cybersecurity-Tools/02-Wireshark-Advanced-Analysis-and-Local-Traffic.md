# Wireshark Masterclass Part 2: Advanced Analysis & Local Traffic

This document builds upon the fundamentals, covering the analysis of local network chatter, modern protocols like QUIC, and the use of Wireshark's powerful statistics tools for high-level investigation.

---

## ### Part 1: Analyzing "Local Chatter"

Not all traffic goes to the internet. Your local network has its own background conversations for device discovery.

#### #### ARP (Address Resolution Protocol)
* **Purpose:** Maps a known IP address to its unknown MAC address on the local network. Essential before IP communication can begin locally.
* **Action:** Used `arp -d *` in an Administrator Command Prompt to clear the cache, started a capture, then immediately pinged the router (`ping [router_IP]`). Filtered for `arp`.
* **Observation:** Captured the two core ARP packets:
    1.  **ARP Request:** Source MAC = Our PC, Destination MAC = **Broadcast** (`ff:ff:ff:ff:ff:ff`), Info = "Who has [Router IP]? Tell [My IP]". Seen by all devices on the local segment.
    2.  **ARP Reply:** Source MAC = Router, Destination MAC = Our PC (**Unicast**), Info = "[Router IP] is at [Router MAC]".
* **Analyst Insight:** ARP traffic reveals the IP-to-MAC mappings of active devices locally. It's the target of **ARP Spoofing** attacks, where an attacker sends false replies to redirect traffic.

#### #### SSDP (Simple Service Discovery Protocol)
* **Purpose:** Used by UPnP (Universal Plug and Play) devices like Smart TVs, printers, and media servers to announce their services and discover others on the local network.
* **How it Works:** Uses UDP Port 1900 and Multicast Address `239.255.255.250`.
* **Analyst Insight:** SSDP traffic is unencrypted and acts as a **reconnaissance goldmine**, revealing the presence, type, brand, and software details of discoverable devices, potentially highlighting vulnerable IoT systems.

#### #### mDNS (Multicast DNS)
* **Purpose:** Acts as a "local DNS," allowing devices to resolve hostnames ending in `.local` (e.g., `irfans-laptop.local`) without needing a central DNS server. Used heavily by Apple's Bonjour.
* **How it Works:** Uses UDP Port 5353 and Multicast Address `224.0.0.251`.
* **Analyst Insight:** Like SSDP, mDNS reveals local hostnames, often including usernames, aiding an attacker in mapping the network and identifying specific user machines.

---

## ### Part 2: Analyzing Modern Protocols - QUIC

**QUIC (Quick UDP Internet Connections)** is a modern transport protocol replacing TCP+TLS for much of the web (especially Google/YouTube).

* **Key Features:** Built on UDP for speed, combines connection/security handshakes (1-RTT or 0-RTT), uses **Connection IDs** for resilience against network changes (e.g., Wi-Fi to cellular), **always encrypted**.
* **Analysis in Wireshark:**
    * **Filter:** `quic`.
    * **Long Headers (`Initial`/`Handshake`):** Seen at the start, establish the connection and contain the **Connection ID**. Crucially, they nest the **TLS Handshake (`CRYPTO` frame)** inside the very first packets.
    * **Short Headers:** Used for subsequent data transfer, much simpler.
    * **Protected Payload:** All application data after the handshake is encrypted and shown as raw bytes (`Remaining Payload`). **Cannot be decrypted passively.**
* **Security Considerations:** While encrypted, potential weaknesses include **0-RTT Replay Attacks** (re-sending encrypted requests like "Transfer $10") and **Protocol Downgrade Attacks** (an MITM blocking QUIC to force a fallback to standard HTTPS).

---

## ### Part 3: Mastering High-Level Analysis Tools (Statistics Menu)

These tools provide rapid overviews of large captures.

#### #### Workflow: Overview -> Isolate -> Deep Dive

#### #### Tool 1: `Statistics > Protocol Hierarchy`
* **Purpose:** Gives a percentage breakdown of all protocols in the capture. Answers "What are people doing?"
* **Example:** In our capture with web browsing and ping, TCP (carrying HTTPS) dominated due to the many packets needed for a website load, while UDP (carrying DNS) and ICMP (carrying Ping) were much smaller percentages.

#### #### Tool 2: `Statistics > Endpoints`
* **Purpose:** Lists all unique IPs (IPv4/IPv6), TCP/UDP ports, and MAC addresses. Shows packet/byte counts (Total, Tx, Rx). Answers "Who is on the network?" and "Who is busiest?"
* **Example:** We identified our local IP (`192.168.100.95`) as the chattiest. We used DNS filters (`dns.resp.name == "..."`) within Wireshark to resolve external IPs, confirming `github.com` used TCP Port 443 (HTTPS) and our DNS server was our router (`192.168.100.1`) using UDP Port 53.

#### #### Tool 3: `Statistics > Conversations`
* **Purpose:** Maps who is talking to whom, listing pairs of endpoints and the traffic flow (packets/bytes A->B and B->A) between them.
* **Example:** We isolated the `ping 8.8.8.8` conversation, seeing 4 packets A->B (requests) and 4 packets B->A (replies).
* **Key Feature:** The **`Right-Click > Apply as Filter`** shortcut instantly creates a display filter to isolate only the packets belonging to the selected conversation, crucial for the "Isolate" step.

#### #### Final Exercise ("Suspicious Connection")
* **Action:** Captured mixed traffic (web + `ping scanme.nmap.org`).
* **Workflow Applied:**
    1.  Used **Protocol Hierarchy** to see TCP dominance.
    2.  Used **Endpoints** to identify our IP.
    3.  Used **Conversations** to find the specific ICMP exchange with `scanme.nmap.org`'s IP (8 packets total).
    4.  Used **Apply as Filter** to isolate those 8 packets.
    5.  Used **Packet Details** to analyze the ICMP packets, confirming they were normal pings.
* **Insight:** Successfully demonstrated the complete analyst workflow, proving the ability to systematically investigate a capture file from overview down to specific packet details. The "suspicion" in a real case would come from the context (why ping that specific server?), not the packets themselves.

---
