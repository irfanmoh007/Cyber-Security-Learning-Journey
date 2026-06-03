# Infinity Shell — TryHackMe

**Difficulty**: Easy
**Category**: Web Forensics / Webshell Analysis
**Date**: June 2026

---

## Honest Reflection

This room was about finding a PHP webshell implanted by an attacker 
on a web server. I had the right instincts from the start and found 
the webshell independently. I got stuck on what to do after finding 
it and searched for a hint — not the answer — and executed the 
solution myself.

**What I did well:**
- Went straight to apache logs without being told — correct instinct
- Found suspicious requests in logs independently
- Located the webshell file independently
- Read the PHP code independently
- Asked for a hint not an answer when stuck
- Executed the solution myself after getting direction

**What I need to learn:**
- What PHP webshells look like and how they work
- How to decode commands from apache log entries
- Base64 encoded commands in web requests

---

## What is a Web Shell?

A webshell is a malicious script uploaded to a web server that gives 
an attacker remote command execution through HTTP requests.

**How it works:**
1. Attacker finds a file upload vulnerability on a website
2. Uploads a malicious PHP file disguised as an image or document
3. Accesses that file through the browser
4. Sends commands via URL parameters
5. Server executes those commands and returns output

**Why PHP is commonly used:**
PHP runs server-side on most web servers. A single line of PHP can 
give an attacker full command execution:

```php

```

This means:
- Attacker sends GET request to the PHP file
- cmd parameter contains a base64 encoded command
- PHP decodes it and executes it as a system command
- Output is returned in the HTTP response

**Why base64 encoding:**
Encoding commands hides them from basic log monitoring and WAFs 
(Web Application Firewalls) that look for suspicious keywords like 
"whoami" or "cat /etc/passwd" in URL parameters.

---

## Investigation Methodology

### Step 1 — Check Apache logs first

```bash
cd /var/log/apache2
ls
cat allthelogspresent in the apache2 some logs doesnt display crucial information
```

Apache logs record every HTTP request made to the web server.
When looking for a webshell, suspicious patterns include:
- Repeated requests to the same unusual file
- Requests with long encoded strings in parameters
- Requests to files in unexpected locations like /img/ folder
- POST or GET requests with base64-looking parameters

**What I found:**
Multiple requests pointing to /img/images.php with encoded 
query parameters. Legitimate image folders do not contain PHP 
files — this was immediately suspicious.

---

### Step 2 — Navigate to the webshell location

```bash
cd /var/www/html/img
ls
```

**Red flag pattern:**
PHP files inside an image directory are almost always malicious.
Legitimate image folders contain .jpg, .png, .gif files only.
Finding images.php inside /img/ is an immediate red flag.

---

### Step 3 — Read the webshell file

```bash
cat images.php
```

**What I found:**
```php

```

This is a classic one-liner webshell. Breaking it down:
- `$_GET['cmd']` — reads a parameter called cmd from the URL
- `base64_decode()` — decodes the base64 encoded value
- `system()` — executes the decoded string as a system command

---

### Step 4 — Find the flag in the logs

Go back to the apache logs and grep for requests to images.php:

```bash
grep "images.php query" 
```

The log entries will show the full URL including the encoded 
cmd parameter. Copy the base64 string from those requests.

Then decode in CyberChef:
gchq.github.io/CyberChef → From Base64

The decoded commands show exactly what the attacker ran on the 
server. The flag is hidden inside those commands.

---

## Full Command Sequence

```bash
# Step 1: Check apache logs for suspicious requests
cd /var/log/apache2
cat other.vhost.access.log1.log

# Step 2: Navigate to suspicious location found in logs
cd /var/www/html/img
ls

# Step 3: Read the webshell
cat images.php

# Step 4: Find encoded commands in logs
grep "images.php" /var/log/apache2/other.vhost.access.log1.log

# Step 5: Copy base64 string from URL parameter
# Decode at gchq.github.io/CyberChef using From Base64
```

---

## Key Concepts Learned

**Apache log location:**
/var/log/apache2/access.log — HTTP requests
/var/log/apache2/error.log — Server errors

**Webshell red flags in logs:**
- PHP file inside image or upload directory
- Repeated requests to same unusual file
- Long encoded strings in URL parameters
- Requests at unusual times or from single IP

**PHP webshell one-liner anatomy:**
```php
system(base64_decode($_GET['query']))
```
- system() = execute OS command
- base64_decode() = decode hidden command
- $_GET['x'] = read from URL parameter

**How to find what attacker ran:**
1. Grep logs for the webshell filename
2. Extract the base64 value from the URL parameter
3. Decode in CyberChef
4. Read the plaintext commands

---

## MITRE ATT&CK Reference

**T1505.003 — Server Software Component: Web Shell**

Attackers use webshells to maintain persistent access to 
compromised web servers. Detection methods:
- Monitor for PHP/ASPX files created in web directories
- Alert on system() and exec() function calls in web logs
- Look for base64 encoded strings in HTTP parameters
- File integrity monitoring on web root directories

---

## Red Flags Summary

| Indicator | What it means |
|-----------|---------------|
| PHP file in /img/ folder | File should not be there |
| system() in PHP file | Command execution capability |
| base64_decode($_GET[]) | Encoded command input |
| Repeated requests with long parameters | Active webshell usage |

---

## What I Did Right This Room

When I got stuck after finding the webshell I asked AI for a 
hint specifically saying "don't tell me the answer." This is 
the correct way to use AI assistance:

- Wrong: "What is the flag for Infinity Shell THM"
- Wrong: "How do I solve this room"
- Right: "I found this suspicious file, I think I need to 
  decode something, can you point me in the right direction 
  without giving me the answer"

Getting a nudge in the right direction and executing the 
solution yourself is valid learning. Copying a flag is not.

