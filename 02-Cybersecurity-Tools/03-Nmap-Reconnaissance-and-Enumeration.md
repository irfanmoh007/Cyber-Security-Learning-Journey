# Nmap Reconnaissance and Enumeration

## What Nmap Is

Nmap is a network discovery and service enumeration tool. It helps answer basic but important questions:

- Which hosts are alive?
- Which ports are open?
- What services are running?
- What versions might those services be?
- What operating system or device type might be present?

For defenders, Nmap is useful because attackers often begin with similar discovery steps. Understanding Nmap output makes it easier to recognize exposed services, unnecessary attack surface, and suspicious scanning behavior in logs.

## Common Use Cases

| Use Case | Why It Matters |
| --- | --- |
| Asset discovery | Identify live hosts on a subnet |
| Port scanning | Find exposed services such as SSH, HTTP, SMB, RDP, or databases |
| Service enumeration | Determine application names and versions |
| Vulnerability validation | Confirm whether a risky service is actually exposed |
| Lab recon | Build an attack path during authorized practice rooms |
| Defensive baselining | Compare expected open ports against actual open ports |

## Scan Types I Should Understand

| Scan Type | Purpose | Analyst Note |
| --- | --- | --- |
| Ping sweep | Find live hosts | May be blocked by firewalls |
| TCP SYN scan | Identify open TCP ports | Common and fast |
| TCP connect scan | Full TCP connection | Easier to log on targets |
| UDP scan | Check UDP services | Slower but important for DNS, SNMP, NTP |
| Version detection | Identify service versions | Useful for risk assessment |
| OS detection | Guess operating system | Not always accurate |
| Script scan | Run NSE scripts | Powerful, but use carefully and only with authorization |

## Example Defensive Questions

When reviewing Nmap results, I should ask:

- Is this service expected on this host?
- Is the port exposed to the right network segment?
- Is the service version outdated?
- Is management access exposed, such as SSH, RDP, WinRM, or admin panels?
- Does the host expose file sharing or database services unnecessarily?
- Would a SIEM alert if this scan occurred in a production network?

## Useful Output Fields

| Field | Meaning |
| --- | --- |
| Host | IP address or hostname being scanned |
| Port | Network port number |
| State | Open, closed, filtered, or unknown |
| Service | Guessed service name |
| Version | Detected product/version, if available |
| Reason | Why Nmap believes the port has that state |

## Detection Opportunities

Nmap activity can create visible patterns:

- Many connection attempts from one source to many ports
- Sequential port scanning
- SYN packets without full application use
- Failed connections across many hosts
- DNS or reverse lookup activity during recon
- Web server logs showing unusual probes

In a SIEM, this can become a rule such as:

```text
Same source IP connects to many destination ports in a short time window.
```

## TryHackMe Advanced Nmap Notes

These are my notes from the Advanced Nmap module. The main lesson is that Nmap is not just "scan this IP." It gives a lot of control over host discovery, DNS behavior, scan type, timing, output, and how much evidence the scan may create in logs.

### Listing Targets Before Scanning

```bash
nmap -sL -n 10.10.12.13/29
```

What I learned:

- `-sL` lists the targets that Nmap would scan, but it does not actually scan them.
- `-n` disables DNS lookups.
- Without `-n`, Nmap may perform reverse DNS lookups, and those DNS lookups can generate logs before any port scan happens.

This is useful when I want to confirm the IP range that Nmap will expand from CIDR notation before starting a real scan.

### Host Discovery Only

By default, Nmap performs host discovery first. It tries to identify live hosts, then it proceeds to scan ports on hosts it thinks are up.

If I only want host discovery and do not want port scanning, I should use:

```bash
nmap -sn TARGET
```

Key point:

- `-sn` means host discovery only.
- If Nmap finds a live host and `-sn` is not used, Nmap can continue into port scanning.

### ARP Discovery on the Same Subnet

When I am on the same local subnet as the target range, ARP discovery is very effective because ARP is used to map IP addresses to MAC addresses on the local network.

```bash
sudo nmap -PR -sn -n 10.200.6.0/24
```

What the flags mean:

- `-PR` uses ARP discovery.
- `-sn` keeps it to host discovery only.
- `-n` disables DNS lookups.

Important correction for my notes: `-PR -n` controls ARP and DNS behavior, but `-sn` is what prevents Nmap from continuing into a port scan.

### ICMP Timestamp Discovery

```bash
sudo nmap -PP -sn TARGET
```

What I learned:

- `-PP` forces ICMP timestamp requests.
- `-sn` keeps the scan focused on host discovery.
- This can be useful when ICMP echo requests are blocked but timestamp replies are still allowed.

### Host Discovery Scan Types

| Scan Type | Example Command | Use Case |
| --- | --- | --- |
| ARP scan | `sudo nmap -PR -sn 10.200.6.0/24` | Best for local subnet discovery |
| ICMP echo scan | `sudo nmap -PE -sn 10.200.6.0/24` | Classic ping-style discovery |
| ICMP timestamp scan | `sudo nmap -PP -sn 10.200.6.0/24` | Tests ICMP timestamp replies |
| ICMP address mask scan | `sudo nmap -PM -sn 10.200.6.0/24` | Tests ICMP address mask replies |
| TCP SYN ping scan | `sudo nmap -PS22,80,443 -sn 10.200.6.0/30` | Checks likely open TCP services |
| TCP ACK ping scan | `sudo nmap -PA22,80,443 -sn 10.200.6.0/30` | Can reveal hosts through firewall behavior |
| UDP ping scan | `sudo nmap -PU53,161,162 -sn 10.200.6.0/30` | Useful for UDP-heavy services like DNS/SNMP |

Reminder:

```text
Use -sn when I only want host discovery without port scanning.
```

### DNS Options

| Option | Purpose |
| --- | --- |
| `-n` | No DNS lookup |
| `-R` | Reverse-DNS lookup for all hosts |
| `-sn` | Host discovery only |

Defensive note: DNS behavior matters. Even when a port scan has not started, DNS lookups can still leave evidence in DNS resolver logs.

### Advanced TCP Scan Types

| Port Scan Type | Example Command | What It Helps Me Learn |
| --- | --- | --- |
| TCP Null scan | `sudo nmap -sN 10.49.146.67` | Sends no TCP flags |
| TCP FIN scan | `sudo nmap -sF 10.49.146.67` | Sends FIN flag |
| TCP Xmas scan | `sudo nmap -sX 10.49.146.67` | Sends FIN, PSH, and URG flags |
| TCP Maimon scan | `sudo nmap -sM 10.49.146.67` | Uses FIN/ACK behavior on some systems |
| TCP ACK scan | `sudo nmap -sA 10.49.146.67` | Helps map firewall filtering, not open ports directly |
| TCP Window scan | `sudo nmap -sW 10.49.146.67` | Similar to ACK scan, but checks TCP window behavior |
| Custom TCP scan | `sudo nmap --scanflags URGACKPSHRSTSYNFIN 10.49.146.67` | Manually sets TCP flags |

Null, FIN, and Xmas scans rely on unusual TCP flag combinations. On some systems, closed ports reply with RST while open or filtered ports may not reply. ACK and Window scans are more useful for understanding firewall filtering than for normal service discovery.

### Spoofing, Decoys, Idle Scan, and Fragmentation

These options are important to understand from a defender perspective because they affect attribution, packet structure, and how traffic appears in logs. They should only be used in authorized labs.

| Technique | Example / Option | Purpose |
| --- | --- | --- |
| Spoofed source IP | `sudo nmap -S SPOOFED_IP 10.49.146.67` | Sets a different source IP where routing allows it |
| Spoofed MAC address | `--spoof-mac SPOOFED_MAC` | Changes the source MAC address used by Nmap |
| Decoy scan | `nmap -D DECOY_IP,ME 10.49.146.67` | Mixes decoy sources with the real scanner |
| Idle scan | `sudo nmap -sI ZOMBIE_IP 10.49.146.67` | Uses a third-party "zombie" host for blind scanning |
| Fragment into 8 bytes | `-f` | Fragments IP data into smaller pieces |
| Fragment into 16 bytes | `-ff` | Uses larger fragmentation than `-f` |

### Extra Packet Control Options

| Option | Purpose |
| --- | --- |
| `--source-port PORT_NUM` | Specify source port number |
| `--data-length NUM` | Append random data to reach the given length |

These options change how packets look on the wire. From a SOC view, this matters because attackers may try to blend in with expected source ports or change packet size patterns.

### Core Port Scan Types

| Port Scan Type | Example Command | Notes |
| --- | --- | --- |
| TCP connect scan | `nmap -sT MACHINE_IP` | Completes a full TCP connection; easier to log |
| TCP SYN scan | `sudo nmap -sS MACHINE_IP` | Half-open scan; common and fast |
| UDP scan | `sudo nmap -sU MACHINE_IP` | Finds UDP services but can be slower |

These scan types are the starting point for discovering running TCP and UDP services.

### Port Selection and Timing

| Option | Purpose |
| --- | --- |
| `-p-` | Scan all ports |
| `-p1-1023` | Scan ports 1 to 1023 |
| `-F` | Scan 100 most common ports |
| `-r` | Scan ports in consecutive order |
| `-T<0-5>` | Timing template; `-T0` is slowest and `-T5` is fastest |
| `--max-rate 50` | Send no more than 50 packets per second |
| `--min-rate 15` | Try to send at least 15 packets per second |
| `--min-parallelism 100` | Use at least 100 probes in parallel |

Timing affects speed, reliability, and detection. Faster scans may finish quickly but are easier to notice. Slower scans may be quieter but take longer and can still be detected by good monitoring.

### Version Detection, OS Detection, Scripts, and Output

| Option | Meaning |
| --- | --- |
| `-sV` | Determine service/version information on open ports |
| `-sV --version-light` | Try the most likely probes |
| `-sV --version-all` | Try all available probes |
| `-O` | Detect operating system |
| `--traceroute` | Run traceroute to the target |
| `--script=SCRIPTS` | Run selected Nmap scripts |
| `-sC` or `--script=default` | Run default scripts |
| `-A` | Equivalent to `-sV -O -sC --traceroute` |
| `-oN` | Save output in normal format |
| `-oG` | Save output in grepable format |
| `-oX` | Save output in XML format |
| `-oA` | Save output in normal, XML, and grepable formats |

### Verbosity and Reasoning

| Option | Purpose |
| --- | --- |
| `--reason` | Explains how Nmap made its conclusion |
| `-v` | Verbose output |
| `-vv` | Very verbose output |
| `-d` | Debugging output |
| `-dd` | More detailed debugging output |

`--reason` is especially useful while learning because it explains why Nmap marked a port as open, closed, filtered, or unfiltered.

### Main Takeaways

- `-sn` is the key flag for host discovery only.
- `-n` prevents DNS lookups and reduces DNS log noise.
- ARP discovery is best when scanning the same subnet.
- ICMP, TCP, and UDP discovery methods behave differently depending on firewall rules.
- Advanced TCP scans teach how flags and firewall behavior affect scan results.
- Timing and rate options change how noisy or fast a scan is.
- Output options matter because saved results are easier to compare, grep, parse, and document.

## How Nmap Fits My Learning Path

Nmap connects networking knowledge with security practice. To use it well, I need to understand ports, TCP flags, services, firewall behavior, and how scans appear from the defender side.
