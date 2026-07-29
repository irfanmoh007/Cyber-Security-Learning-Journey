# TryHackMe: Webserver Attacks

This document details common web server misconfigurations, vulnerability patterns, and attack vectors observed across different web server technologies (Apache, Python HTTP, Node.js, and Nginx).

---

## Patterns That Apply Everywhere

Looking back at all four web servers, the same categories of misconfiguration appear repeatedly:

| Misconfiguration | Apache | Python HTTP | Node.js | Nginx |
| :--- | :--- | :--- | :--- | :--- |
| **Version disclosure in headers** | Yes | Yes | Partial | Yes |
| **Directory listing** | `/files/` | Root path | N/A | `/files/` |
| **Exposed status or debug endpoint** | `/server-status` | N/A | `/api/debug/env`, `/api/routes` | `/nginx_status` |
| **Sensitive files accessible** | `backup.bak`, `internal-notes.txt` | `.env`, `notes.txt`, `backup.zip` | `config.js` | `server-config.txt`, `deploy-notes.txt` |
| **Missing security headers** | All | All | All | All |

---

## Analysis of Common Web Server Misconfigurations

The consistent thread is that **default configurations prioritise ease of deployment over security**. Version disclosure, directory listings, and status pages are enabled by default for diagnostic purposes. They make the administrator's job easier. Removing or restricting them takes deliberate action. 

In real engagements, finding these patterns does not indicate negligence. It indicates that the default settings were never reviewed.

### 1. Version Disclosure in Headers
Web servers often leak their exact software name and version number in the `Server` HTTP response header. 
*   **Security Risk:** Attackers use this to identify the exact version running and look up publicly available exploits (CVEs).
*   **Remediation:** Disable version tokens in config files (e.g., set `ServerTokens Prod` in Apache, or `server_tokens off;` in Nginx).

### 2. Directory Listing (Directory Indexing)
When a user requests a directory path that does not contain an index file (like `index.html`), some web servers automatically list all files in that directory.
*   **Security Risk:** Exposes the directory structure and files, allowing attackers to download sensitive configuration assets or code files.
*   **Remediation:** Disable directory listing (e.g., remove `Indexes` from `Options` directive in Apache, or set `autoindex off;` in Nginx).

### 3. Exposed Status/Debug Endpoints
Diagnostic endpoints are frequently left open without authentication.
*   **Security Risk:** Endpoints like `/server-status` (Apache), `/nginx_status` (Nginx), or Node.js environment paths leak sensitive server stats, active connection IPs, routes, and internal environment variables.
*   **Remediation:** Restrict access to these endpoints by IP address (localhost only) and require authentication.

### 4. Sensitive Files Accessible in Root
Accidental storage of backups, notes, and environment configurations in the public web root.
*   **Security Risk:** Files like `.env`, `backup.zip`, or `notes.txt` contain API keys, database credentials, and deployment details that can be read directly by anyone via the browser.
*   **Remediation:** Never store configuration files, credentials, or backups in the web root. Use `.gitignore` to prevent committing them to production.
