# Blind SQL Injection with Time Delays and Information Retrieval

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | Blind SQL injection with time delays and information retrieval |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Blind SQL Injection vulnerability using time delays to retrieve sensitive database information.

Unlike traditional SQL Injection vulnerabilities, the application did not display:
- database errors
- query results
- conditional response differences

Instead, the attacker identified a vulnerable injection point inside the `TrackingId` cookie and used database sleep functions to measure server response times.

By injecting SQL conditions that intentionally delayed the response, it became possible to:
- confirm SQL Injection
- identify password length
- extract the administrator password character-by-character

The PostgreSQL function used in this lab was:

```sql
pg_sleep()
```

---

# Vulnerability Type

- Blind SQL Injection (Blind SQLi)
- Time-Based SQL Injection
- Conditional Time Delay SQL Injection

---

# Impact

An attacker can extract sensitive information from the database even when:
- no errors are displayed
- no query results are visible
- no response differences exist

This may lead to:
- credential disclosure
- unauthorized administrator access
- database enumeration
- sensitive data exposure
- complete database compromise

Time-based Blind SQL Injection vulnerabilities remain highly critical despite minimal visible feedback.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Pets** category.
3. Capture the request using Burp Suite.
4. Identify the vulnerable `TrackingId` cookie.
5. Test whether time-based SQL Injection is possible using:

```sql
TrackingId='||pg_sleep(10)||';
```

6. Observe that the server response is delayed by 10 seconds.
7. Confirm the injection point inside the `TrackingId` cookie.
8. Test conditional time delays using:

```sql
TrackingId='|| CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END ||';
```

9. Observe that the response delays for 10 seconds because the condition evaluates to TRUE.
10. Determine the administrator password length using:

```sql
TrackingId='|| CASE WHEN ((LENGTH((SELECT password FROM users WHERE username='administrator'))) = 1) THEN pg_sleep(10) ELSE pg_sleep(0) END ||';
```

11. Send the payload to Burp Suite Intruder.
12. Extract password characters using:

```sql
TrackingId='|| CASE WHEN ((SUBSTR((SELECT password FROM users WHERE username='administrator'),$1$,1)) = '$a$') THEN pg_sleep(10) ELSE pg_sleep(0) END ||';
```

13. Configure Intruder payload positions:
   - Position 1 → numbers from 1–20
   - Position 2 → brute-force characters

14. Sort Intruder results by highest response time.
15. Identify characters causing delayed responses.
16. Reconstruct the administrator password.
17. Login using the extracted administrator credentials.
18. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appeared to be a normal e-commerce website containing product filtering functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Time%20Delays%20and%20Information%20Retrieval/screenshots/lab10.png?raw=true)

**Caption:** Initial application interface containing the Pets category filter.

---

## Step 2 — Identifying the Injection Point

The Pets category request was intercepted using Burp Suite.

A payload was injected into the `TrackingId` cookie:

```sql
TrackingId='||pg_sleep(10)||';
```

The server response was delayed by 10 seconds, confirming that the `TrackingId` cookie was vulnerable to SQL Injection.

![Time Delay Confirmation](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Time%20Delays%20and%20Information%20Retrieval/screenshots/lab10(1).png?raw=true)

**Caption:** Confirming the SQL Injection point using a time-delay payload.

---

## Step 3 — Testing Conditional Time Delays

The following conditional payload was used:

```sql
TrackingId='|| CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END ||';
```

Because the condition `1=1` evaluated to TRUE, the server response delayed for 10 seconds.

This confirmed successful execution of conditional SQL statements.

![Conditional Delay](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Time%20Delays%20and%20Information%20Retrieval/screenshots/lab10(2).png?raw=true)

**Caption:** Testing conditional SQL execution using time delays.

---

## Step 4 — Determining the Password Length

The following payload was crafted to identify the administrator password length:

```sql
TrackingId='|| CASE WHEN ((LENGTH((SELECT password FROM users WHERE username='administrator'))) = 1) THEN pg_sleep(10) ELSE pg_sleep(0) END ||';
```

The response delay indicated whether the tested password length was correct.

![Password Length Enumeration](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Time%20Delays%20and%20Information%20Retrieval/screenshots/lab10(3).png?raw=true)

**Caption:** Creating a payload to identify the administrator password length.

---

## Step 5 — Preparing the Character Extraction Payload

The payload was modified to extract password characters one-by-one:

```sql
TrackingId='|| CASE WHEN ((SUBSTR((SELECT password FROM users WHERE username='administrator'),$1$,1)) = '$a$') THEN pg_sleep(10) ELSE pg_sleep(0) END ||';
```

Burp Suite Intruder was configured with:
- Position 1 → character position
- Position 2 → brute-force character testing

![Substring Payload](screenshots/blind-time-based-5.png)

**Caption:** Preparing substring-based payloads for password extraction.

---

## Step 6 — Configuring Intruder Payload Positions

Intruder payload positions were configured:
- Position 1 → numbers from 1 to 20
- Position 2 → brute-force characters

![Intruder Configuration](screenshots/blind-time-based-6.png)

**Caption:** Configuring Burp Suite Intruder payload positions.

---

## Step 7 — Extracting Password Characters

The Intruder attack was executed.

The results were sorted using the **Response Received** column:
- requests with the highest response time indicated correct characters

Each valid character was noted sequentially inside a text editor until the full administrator password was reconstructed.

![Intruder Results](screenshots/blind-time-based-7.png)

**Caption:** Identifying valid password characters through delayed responses.

---

## Step 8 — Logging into the Administrator Account

The extracted administrator credentials were used on the login page.

![Administrator Login](screenshots/blind-time-based-8.png)

**Caption:** Attempting login using extracted administrator credentials.

---

## Step 9 — Successful Administrator Access

The administrator login succeeded successfully.

The lab was solved successfully.

![Administrator Access](screenshots/blind-time-based-9.png)

**Caption:** Successful administrator login after extracting credentials via Blind SQL Injection.

---

# Why It Works

The application directly inserted user-controlled cookie data into backend SQL queries without proper sanitization.

The attacker used:
- PostgreSQL sleep functions
- conditional SQL logic
- response timing analysis
- automated brute-force extraction

to infer sensitive database information.

The payload:

```sql
CASE WHEN (condition) THEN pg_sleep(10)
```

caused measurable server delays whenever the condition evaluated to TRUE.

This allowed extraction of:
- password length
- individual password characters

without directly displaying database results.

---

# Security Risk

This vulnerability can result in:
- credential disclosure
- database enumeration
- unauthorized administrator access
- sensitive data exposure
- complete database compromise

Time-based Blind SQL Injection vulnerabilities remain extremely dangerous even when applications suppress all visible errors.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Apply strict input validation.
- Securely handle cookies and user-controlled headers.
- Restrict database permissions using least privilege principles.

---

# Learning Section

## What I Learned

- Blind SQL Injection can extract data using response timing differences.
- PostgreSQL's `pg_sleep()` function can create intentional delays.
- Time-based SQL Injection works even when applications hide all errors and query results.
- Burp Suite Intruder can automate password extraction attacks.
- SQL functions such as `LENGTH()` and `SUBSTR()` help retrieve hidden database values.

---

# Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy
- PortSwigger SQL Injection Cheat Sheet

---

# Conclusion

This lab demonstrated a time-based Blind SQL Injection vulnerability through the `TrackingId` cookie.

By leveraging PostgreSQL sleep functions and conditional SQL logic, sensitive administrator credentials were extracted character-by-character without directly displaying database results, ultimately allowing unauthorized administrator access.
