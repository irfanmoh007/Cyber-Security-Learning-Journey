# Snort and Zeek Network Detection

## What They Are

Snort and Zeek are network security monitoring tools, but they think differently.

| Tool | Main Role | Output Style |
| --- | --- | --- |
| Snort | Intrusion Detection System (IDS) | Alerts based on signatures/rules |
| Zeek | Network Security Monitor (NSM) | Rich protocol logs and connection metadata |

Snort is good at saying "this traffic matched a known suspicious pattern." Zeek is good at saying "this is what happened on the network."

## Snort Use Cases

- Detect known attack patterns
- Alert on suspicious payloads or protocol behavior
- Test IDS signatures in a lab
- Learn how rule logic works
- Validate whether traffic would trigger an IDS alert

## Snort Rule Concepts

| Concept | Meaning |
| --- | --- |
| Action | What Snort does, such as alert or drop |
| Protocol | TCP, UDP, ICMP, or IP |
| Source and destination | Direction of the traffic |
| Ports | Services involved |
| Message | Alert text shown to analysts |
| Content match | Pattern Snort searches for |
| SID | Unique rule identifier |

Example structure:

```text
alert tcp any any -> any 80 (msg:"Suspicious HTTP traffic"; content:"example"; sid:1000001;)
```

## Zeek Use Cases

- Build a timeline of network activity
- Review DNS, HTTP, SSL/TLS, SMB, SSH, and connection logs
- Investigate beaconing or suspicious outbound traffic
- Enrich SIEM alerts with protocol-level evidence
- Detect unusual patterns without needing full packet payloads

## Important Zeek Logs

| Log | What It Shows |
| --- | --- |
| conn.log | Network connections and session metadata |
| dns.log | DNS queries and answers |
| http.log | HTTP requests, hosts, methods, and user agents |
| ssl.log | TLS metadata, certificates, and SNI |
| files.log | Files observed in network traffic |
| notice.log | Zeek-generated notices |
| weird.log | Unusual protocol behavior |

## Snort vs Zeek

| Question | Better Tool |
| --- | --- |
| Did this traffic match a known malicious pattern? | Snort |
| What domains did this host query? | Zeek |
| What HTTP user agents appeared? | Zeek |
| Did a known exploit signature fire? | Snort |
| What was the timeline of connections? | Zeek |
| What should I send into a SIEM for hunting? | Zeek logs plus Snort alerts |

## Detection Engineering Value

Snort helps me learn signature-based detection. Zeek helps me learn behavior-based investigation. Together they show why defenders need both alerting and context.

Good SOC investigations rarely stop at an alert. A Snort alert can tell me what matched; Zeek logs can help me understand what happened before and after.
