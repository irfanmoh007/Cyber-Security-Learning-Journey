# Wireshark Deep Dive: Practical Traffic Analysis

This document details the practical analysis of live network traffic using Wireshark. It moves beyond theory to cover the hands-on skills required to dissect conversations, understand packet anatomy, and identify different types of protocols in the wild.

---

## ## Part 1: How Wireshark *Actually* Works

**What problem does it solve?** Wireshark allows us to see the invisible conversations our computer is having. It doesn't capture packets itself; it instructs your computer's network hardware to make a copy of every packet it sees.

* **Wired (Ethernet):** Wireshark enables **"promiscuous mode,"** telling the network card to capture all packets on the wire.
* **Wireless (Wi-Fi):** Wireshark enables **"monitor mode,"** turning your Wi-Fi card into a radio scanner that listens to all packets in the air on a specific channel.

**A Critical Note on Modern Networks:** On a modern, secure (WPA2/WPA3) and switched network, you **cannot** read the content of another user's traffic.
1.  **Switching:** Your router acts as a smart switch, delivering packets *only* to their intended recipient. Your computer never sees the traffic between the router and another device.
2.  **Encryption:** Even if you could see the packets in monitor mode, the data payload is encrypted, making it unreadable without the key.

The primary purpose of Wireshark for an analyst is to master the analysis of a system you control: your own.

---

## ## Part 2: The Analyst's Workflow - A Live HTTPS Dissection

This is the step-by-step process of capturing and analyzing a secure web browsing session.

### ### Step 1: Capturing and Filtering
The most important skill in Wireshark is filtering. A capture of just 10 seconds can yield thousands of packets. We use **display filters** to isolate the one conversation we care about.

**Real-World Example: Visiting `https://example.com`**
1.  **Start Capture:** Begin capturing on your main network interface.
2.  **Generate Traffic:** Open a browser and go to `https://example.com`.
3.  **Stop Capture:** Stop the capture once the page loads.
4.  **Find the IP (DNS Filter):** In the display filter, type `dns`. Find the DNS response that shows the IP address for `example.com`.
5.  **Isolate the Conversation:** Clear the `dns` filter. Use the IP address you found in a new filter: `ip.addr == [server's IP address]`. This shows you only the packets to and from that server.

### ### Step 2: The TCP Handshake (The "Hello")
The first three packets of the conversation establish the connection.
* **Packet 1: `[SYN]`** -> Your computer to the server. ("Hello, are you there?")
* **Packet 2: `[SYN, ACK]`** <- The server to your computer. ("Yes, I'm here and I hear you!")
* **Packet 3: `[ACK]`** -> Your computer to the server. ("Great, connection established.")

### ### Step 3: The TLS Handshake (The "Secret Handshake" 🔒)
Immediately after the TCP handshake, the security handshake begins to create the encrypted tunnel.
* **`Client Hello`:** Your browser sends this message, containing the TLS versions and **Cipher Suites** (encryption plans) it supports. It also includes the **`key_share`** extension—its "public ingredient" for creating the secret key.
* **`Server Hello`:** The server chooses the best cipher suite they both support and sends back its own **`key_share`**. At this point, both sides can independently calculate the **same secret session key**.
* **`Application Data` (from Server):** In modern TLS 1.3, the server immediately sends its **digital certificate** inside an encrypted `Application Data` packet.

### ### Step 4: The HTTP Request (The "Order")
Once the secure tunnel is built, your browser sends the actual request for the webpage.
* **`Application Data` (from Client):** This encrypted packet contains the HTTP request, such as `GET / HTTP/1.1`.

---

## ## Part 3: Mastering the Tool - The Packet Details Pane

The middle pane in Wireshark is the analyst's microscope. It shows a packet broken down into its layers (the OSI model in action).

### ### Layer 1: Frame (Wireshark's Metadata)
* **Analogy:** The post office's internal tracking slip.
* **Key Fields:**
    * `Frame Length`: The total size of the packet on the wire.
    * `Encapsulation type`: The network type. It often shows **Ethernet** even on Wi-Fi because your Wi-Fi driver "translates" the packet into a standard format before the OS sees it.

### ### Layer 2: Ethernet (The Local Hop)
* **Analogy:** The specific gate number for your next flight.
* **Key Fields:**
    * `Destination MAC`: The hardware address of the next device (e.g., your router).
    * `Source MAC`: The hardware address of your computer.

### ### Layer 3: Internet Protocol (The Global Journey 🌐)
* **Analogy:** The final destination airport address on your plane ticket.
* **Key Fields:**
    * `Source Address`: Your computer's **private IP** (e.g., `192.168.100.95`). This is later rewritten to your public IP by your router via **NAT**.
    * `Destination Address`: The server's public IP address.
    * `Time to Live (TTL)`: A hop limit that prevents packets from looping forever. This is a key tool for **OS Fingerprinting** (Windows often uses TTL=128, Linux/macOS uses TTL=64). It is also the mechanism that powers the `traceroute` command.

### ### Layer 4: Transmission Control Protocol (The Conversation Manager 📦)
* **Analogy:** The rules for a registered mail delivery.
* **Key Fields:**
    * `Source Port`: A random, high-numbered "ephemeral" port from your PC.
    * `Destination Port`: A "well-known" port on the server (e.g., `80` for HTTP, `443` for HTTPS).
    * `Sequence (Seq) & Acknowledgment (Ack) Numbers`: The "page numbers" that ensure data arrives reliably and in the correct order.
    * `Flags`: Control signals for the conversation (e.g., `[PSH]` to push data immediately).
    * `Window size value`: Used for **flow control**, telling the other side how much data it's ready to receive.

### ### Layer 7: Application Data (The Letter ✉️)
This is the actual payload. For an encrypted packet, this will just be labeled `Encrypted Application Data`. For an unencrypted HTTP packet, this will be the `Hypertext Transfer Protocol` layer, where you can read the `GET` request and headers in plain text.

---

## ## Part 4: Analyzing "Local Chatter" and Modern Protocols

Not all traffic is for the internet. Your network is constantly "chatting" with itself to discover devices.

* **ARP (Address Resolution Protocol):** Shouts to the local network: "Who has the router's IP address? Tell me your MAC address!" This is essential for local communication and is the target of **ARP Spoofing** attacks.
* **SSDP (Simple Service Discovery Protocol):** Used by media devices to find each other (e.g., your PC finding a Smart TV). An attacker can listen to this to map out all IoT devices on your network.
* **mDNS (Multicast DNS):** The "local phonebook" that lets you use names like `irfans-laptop.local` instead of IP addresses. It reveals the hostnames and usernames of devices on the network.
* **QUIC (Quick UDP Internet Connections):** A modern, faster replacement for TCP+TLS used by Google/YouTube. It's built on UDP and is **always encrypted**. In Wireshark, its encrypted data appears as **"Protected Payload."**
