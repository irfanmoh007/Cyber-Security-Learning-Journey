# Active Directory and Enterprise Security

This document outlines the fundamentals of Active Directory (AD) in enterprise environments, standard Windows auditing protocols, common AD security attack vectors (such as Kerberoasting and Pass-the-Hash), and how security analysts monitor AD directories for compromises.

---

## Part 1: Active Directory Fundamentals

Active Directory Domain Services (AD DS) is Microsoft's directory service, acting as the centralized identity and access management hub for enterprise Windows networks.

### Key Components
*   **Domain Controller (DC):** The server that hosts the Active Directory database (`ntds.dit`) and handles all authentication and authorization requests.
*   **Objects:** Elements within AD, categorized as **Users**, **Groups**, **Computers**, or **Organizational Units (OUs)**.
*   **Active Directory Schema:** Defines the rules, object classes, and attributes that can be created in the AD database.

### Protocols in Action
*   **LDAP (Lightweight Directory Access Protocol):** Used to query and manage directory objects (e.g., searching for all users in the "HR" group).
*   **Kerberos:** The primary authentication protocol in Windows domains. It uses a ticketing system based on a trusted third party—the Key Distribution Center (KDC)—running on the DC.
*   **DNS (Domain Name System):** The backbone of AD. Devices use DNS to locate Domain Controllers (via SRV records) and resolve local resource hostnames.

---

## Part 2: How Kerberos Authentication Works

Kerberos uses symmetric cryptography to authenticate clients to services without sending passwords over the network.

```text
  Client          1. AS-REQ (I want to log in) ────────►   Key Distribution
          ◄──────── 2. AS-REP (Here is your TGT)       Center (KDC)
                  
  Client          3. TGS-REQ (Request Service Ticket) ──►  Key Distribution
          ◄──────── 4. TGS-REP (Here is your Service)  Center (KDC)
                  
  Client          5. AP-REQ (Present Ticket) ──────────►   Application Server
```

1.  **Authentication Service Exchange (AS):** The client requests a Ticket Granting Ticket (TGT). The KDC verifies the user's identity and returns an encrypted TGT.
2.  **Ticket Granting Service Exchange (TGS):** The client presents the TGT to request a service ticket for a specific service (like access to a file share or database). The KDC returns a service ticket encrypted with the service account's password hash.
3.  **Application Service Exchange (AP):** The client presents the service ticket to the application server to gain access.

---

## Part 3: Common Active Directory Attacks

Attackers target AD to escalate privileges and gain control over the entire enterprise network.

### 1. Kerberoasting
*   **Mechanism:** Any authenticated domain user can request a Kerberos service ticket (TGS) for any service account with a Service Principal Name (SPN). Because the KDC encrypts the service ticket using the service account's password hash, the attacker can request the ticket, extract it from their local computer's memory, and crack the hash offline using tools like Hashcat or John the Ripper.
*   **Impact:** If the service account has a weak password and domain admin privileges, the attacker gains domain admin control.

### 2. Pass-the-Hash (PtH)
*   **Mechanism:** Windows uses NTLM hashes for legacy authentication. If an attacker gains local administrator rights on a workstation, they can dump the NTLM hashes of recently logged-in users from the LSASS memory space. The attacker can then use the raw NTLM hash to authenticate to other computers on the network *without* decrypting it into the cleartext password.

### 3. Golden Ticket Attack
*   **Mechanism:** The Active Directory Kerberos service relies on a special account named `krbtgt`. If an attacker compromises a Domain Controller and extracts the NTLM hash of the `krbtgt` account, they can forge their own Ticket Granting Tickets (TGTs).
*   **Impact:** The attacker can grant themselves Domain Admin privileges and forge tickets that are valid for years, providing permanent, undetectable backdoor access to the entire forest.

---

## Part 4: Auditing and Monitoring AD Events

Security analysts monitor Domain Controller Event Logs for anomalous activity. Critical Event IDs include:

*   **Event ID 4720:** A user account was created (monitored to detect unauthorized backdoor accounts).
*   **Event ID 4728 / 4732 / 4756:** A member was added to a security-enabled global, local, or universal group (critical for detecting privilege escalation, such as adding a user to "Domain Admins").
*   **Event ID 4768:** A Kerberos Ticket Granting Ticket (TGT) was requested (analyzed for anomalous login locations or massive bursts indicating brute-force attacks).
*   **Event ID 4769:** A Kerberos Service Ticket (TGS) was requested (essential for detecting **Kerberoasting** activity when a user requests an unusually high number of service tickets).
