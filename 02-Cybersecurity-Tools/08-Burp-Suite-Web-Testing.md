# Burp Suite Web Application Testing

## What Burp Suite Is

Burp Suite is a web application security testing platform. It sits between the browser and the web application so I can inspect, modify, replay, and analyze HTTP requests and responses.

Burp is useful for understanding how web applications actually communicate, not just how they look in the browser.

## Main Components

| Component | Purpose |
| --- | --- |
| Proxy | Intercept browser requests and responses |
| Repeater | Modify and resend individual requests |
| Intruder | Automate controlled payload testing |
| Decoder | Decode or encode data formats |
| Comparer | Compare two requests or responses |
| Logger/HTTP history | Review traffic generated during testing |

## Use Cases

- Understand login flows and session cookies
- Inspect hidden parameters
- Test authorization issues such as IDOR
- Replay requests to confirm behavior
- Analyze API endpoints
- Study request methods, headers, and response codes
- Practice web vulnerability labs safely

## Web Vulnerability Areas Burp Helps With

| Area | What to Look For |
| --- | --- |
| Authentication | Weak login or reset flows |
| Authorization | Accessing another user's object or admin function |
| Input validation | Injection, file upload, or parsing weaknesses |
| Session management | Cookie flags, token handling, logout behavior |
| API security | Exposed endpoints, weak object-level authorization |
| Information disclosure | Error messages, metadata, hidden routes |

## Analyst Mindset

When using Burp, I should ask:

- What request is the browser really sending?
- Which parameters control access?
- Is the server validating authorization, or only the frontend?
- Can object IDs be changed?
- Are tokens random and protected?
- Does the response reveal too much?
- Would this request be logged by a WAF, proxy, or SIEM?

## Defensive Value

Even though Burp is often used for web testing, it helps defenders too. Understanding request-level attacks makes it easier to:

- Read web server logs
- Investigate suspicious API activity
- Write better detection logic
- Understand how IDOR, upload abuse, and weak reset flows appear in real traffic

## Safety Note

Burp should only be used on applications I own, lab targets, intentionally vulnerable apps, or environments where testing is explicitly authorized.
