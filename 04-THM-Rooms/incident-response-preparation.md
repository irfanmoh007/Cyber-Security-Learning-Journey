# Incident Response: Preparation — TryHackMe

**Difficulty**: Easy/Medium
**Category**: Incident Response / Blue Team
**Date**: June 2026

---

## Honest Reflection

This room introduced the foundational concepts of incident 
response with a focus on the preparation phase. Unlike 
technical rooms with hands-on tools, this was a conceptual 
room that covers how organisations structure themselves 
to respond to incidents effectively.

**Key insight:**
Most cybersecurity students focus entirely on technical 
skills. But in real SOC work, preparation — having the 
right people, policies, and tools in place BEFORE an 
incident — determines whether a company survives an 
attack or gets destroyed by one.

---

## What is Incident Response?

Incident Response (IR) is a structured approach to 
handling security incidents with the goal of:
- Minimising the impact of the incident
- Reducing recovery time
- Lowering operational costs
- Preventing recurrence

**The key word is structured.**
Without a plan, organisations panic during incidents 
and make expensive mistakes. With a plan, every person 
knows exactly what to do the moment an incident is 
confirmed.

---

## The Incident Response Process — PICERL

The industry standard IR lifecycle has six phases:
┌─────────────────┐

│  1. Preparation │ ← This room focuses here

└────────┬────────┘

↓

┌─────────────────────┐

│  2. Identification  │

└────────┬────────────┘

↓

┌──────────────────┐

│  3. Containment  │

└────────┬─────────┘

↓

┌──────────────────────┐

│  4. Eradication      │

└────────┬─────────────┘

↓

┌──────────────────┐

│  5. Recovery     │

└────────┬─────────┘

↓

┌──────────────────────┐

│  6. Lessons Learned  │

└──────────────────────┘

| Phase | What happens |
|-------|-------------|
| Preparation | Build teams, policies, tools before incidents |
| Identification | Detect and confirm an incident occurred |
| Containment | Stop the spread — isolate affected systems |
| Eradication | Remove the threat completely from environment |
| Recovery | Restore systems to normal operation |
| Lessons Learned | Document what happened and improve defenses |

---

## Phase 1 — Preparation in Detail

Preparation has three pillars:

### Pillar 1 — People

**Awareness and Education:**
Employees are the first line of defense. They need 
regular training on:
- Current threats targeting the organisation
- How to recognise phishing emails
- What to do when they suspect an incident
- Who to contact when something looks wrong

**Security awareness is not a one-time training.**
Threats evolve constantly. Awareness programs must 
be continuous and updated regularly with new attack 
techniques.

---

**CSIRT — Cybersecurity Incident Response Team:**

Every organisation needs a dedicated team trained 
and ready to respond to incidents. A proper CSIRT 
contains members from multiple disciplines:

| Role | Responsibility |
|------|---------------|
| Technical experts | Investigate and contain the threat |
| Business representatives | Assess operational impact |
| Legal counsel | Handle legal obligations and liability |
| Public relations | Manage external communications |
| Management | Make high-level decisions |

**Why non-technical members matter:**
A breach is not just a technical problem. It has 
legal implications (GDPR, data breach notifications), 
business implications (operations, finances), and 
reputational implications (customer trust, media).
The CSIRT handles all of these simultaneously.

The team may also be called:
- CIRT — Computer Incident Response Team
- CERT — Computer Emergency Response Team

---

### Pillar 2 — Policies and Documentation

Having the right policies in place before an incident 
means responders can act immediately without waiting 
for approvals or making decisions under pressure.

**Key documents every organisation needs:**

**Incident Response Policy:**
The high-level document that defines:
- What constitutes an incident
- Who has authority to declare an incident
- What the organisation's obligations are
- Escalation procedures

**Response Procedures (Playbooks):**
Step-by-step guides for specific incident types.
A playbook for ransomware looks different from a 
playbook for a data breach or a DDoS attack.

Good playbooks answer:
- Who do you call first?
- Which systems do you isolate?
- What evidence do you preserve?
- When do you notify customers?
- When do you contact law enforcement?

**Why documentation must be strict and detailed:**
During an incident, people are under extreme stress 
and time pressure. Vague guidelines lead to mistakes. 
Detailed procedures mean anyone on the team can 
follow the steps correctly even in a crisis.

**Documentation also enables:**
- Consistency across incidents
- Training new team members
- Legal defensibility after an incident
- Continuous improvement through lessons learned

---

### Pillar 3 — Technology

**Asset Management:**
You cannot protect what you do not know exists.
Complete inventory of:
- All endpoints (laptops, desktops, servers)
- All network devices (routers, switches, firewalls)
- All cloud resources
- All software and applications

**Integrated Security Stack:**
All security tools must feed telemetry into a central 
system so analysts have one place to investigate:
Endpoints (Sysmon/EDR)

↓

Network (Firewall/IDS logs)

↓

Applications (Web server logs)

↓

SIEM (centralised telemetry)

↓

SOC Analyst (single pane of glass)

Without integration, responders waste critical time 
jumping between disconnected tools during an incident.

---

**The Jump Bag:**

A jump bag is a pre-prepared kit that every incident 
responder must have ready to go at all times. Named 
after military "go bags" — everything you need, 
ready to deploy immediately.

**Why jump bags matter:**
When an incident is confirmed, every minute counts. 
Spending time installing tools or finding equipment 
costs the organisation money and allows the attacker 
more time to cause damage.

**Typical jump bag contents:**

| Item | Purpose |
|------|---------|
| Forensic laptop | Pre-configured investigation machine |
| Write blockers | Copy drives without modifying evidence |
| Blank hard drives | Store forensic images |
| Bootable USB drives | Boot into forensic OS on victim machine |
| Network cables | Connect to isolated networks |
| Network tap | Capture traffic without alerting attacker |
| Documentation | Checklists and playbooks in printed form |
| Legal forms | Chain of custody documentation |

**Write blockers are critical:**
If you connect a victim drive without a write blocker 
and Windows auto-mounts it, you have potentially 
modified evidence. Write blockers prevent ANY writes 
to the drive — preserving forensic integrity.

---

## Why Preparation is the Most Important Phase

A common mistake is treating incident response as 
something you figure out when an incident happens.

**The reality:**
By the time an incident is confirmed, it is too late 
to build your team, write your playbooks, or set up 
your tools. The attacker is already inside.

**Preparation determines the outcome:**

| Without Preparation | With Preparation |
|--------------------|-----------------|
| Panic and confusion | Structured response |
| No clear authority | Clear escalation path |
| Evidence destroyed | Evidence preserved |
| Long recovery time | Faster containment |
| Expensive mistakes | Controlled response |
| Regulatory failures | Compliance maintained |

---

## Key Concepts Summary

**PICERL** — the six phases of incident response:
Preparation, Identification, Containment, Eradication, 
Recovery, Lessons Learned

**CSIRT** — dedicated response team with technical, 
business, legal, and PR members working together

**Playbooks** — detailed step-by-step procedures for 
specific incident types written before incidents happen

**Jump bag** — pre-configured kit with all tools and 
documentation needed to immediately start responding

**Integrated telemetry** — all security tools feeding 
into one central SIEM so responders have full visibility

---

## Connection to Previous Rooms

This room provides the framework that all previous 
technical rooms fit into:

| Previous Room | IR Phase |
|--------------|---------|
| Splunk/Zeek setup | Preparation — technology |
| SOC Home Lab | Preparation — technology |
| Boogeyman 1-3 | Identification and Analysis |
| SigHunt/Aurora | Preparation — detection rules |
| Sneaky Patch | Identification — finding persistence |

Every technical skill learned so far is a preparation 
activity — building the capability to detect and 
respond when a real incident happens.

---

## Real World Application

Entry level SOC analysts are directly involved in 
preparation activities:
- Maintaining and updating detection rules
- Documenting investigation procedures
- Testing and validating security tool integrations
- Participating in tabletop exercises
- Updating asset inventories

Understanding the full IR lifecycle means you can 
explain to an interviewer not just what you did 
technically but WHY it matters in the bigger picture.
