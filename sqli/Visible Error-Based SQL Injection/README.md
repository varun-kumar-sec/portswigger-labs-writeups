# Visible Error-Based SQL Injection

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | Visible error-based SQL injection |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Visible Error-Based SQL Injection vulnerability where sensitive database information was exposed through database error messages.

The application contained a vulnerable `TrackingId` cookie that directly interacted with backend SQL queries.

Unlike Blind SQL Injection labs:
- the application displayed visible database errors
- attacker-controlled queries triggered informative error messages
- sensitive data could be leaked directly inside the error output

Using a specially crafted SQL Injection payload, the administrator password was extracted from the database and displayed inside the application's error response.

---

# Vulnerability Type

- SQL Injection (SQLi)
- Error-Based SQL Injection
- Information Disclosure

---

# Impact

An attacker can abuse database error messages to extract sensitive information such as:
- usernames
- passwords
- database structures
- internal query behavior

This may lead to:
- credential disclosure
- unauthorized administrator access
- database enumeration
- sensitive data exposure
- complete database compromise

Visible error-based SQL Injection vulnerabilities are highly dangerous because they leak backend information directly to attackers.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Lifestyle** category.
3. Capture the request using Burp Suite.
4. Identify the vulnerable `TrackingId` cookie.
5. Using the PortSwigger SQL Injection cheat sheet, craft the following payload:

```sql
TrackingId=' AND CAST((SELECT password FROM users LIMIT 1) AS bool)--;
```

6. Send the request.
7. Observe that the application returns a database error containing the administrator password.
8. Copy the extracted credentials.
9. Navigate to the login page.
10. Authenticate using the extracted administrator credentials.
11. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appeared to be a normal e-commerce website containing a **Lifestyle** category filter.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Visible%20Error-Based%20SQL%20Injection/screenshots/lab11.png?raw=true)

**Caption:** Initial application interface containing the Lifestyle category filter.

---

## Step 2 — Capturing the Request

The Lifestyle category request was intercepted using Burp Suite.

During inspection, the vulnerable `TrackingId` cookie was identified.

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Visible%20Error-Based%20SQL%20Injection/screenshots/lab11(1).png?raw=true)

**Caption:** Capturing the request and identifying the vulnerable TrackingId cookie.

---

## Step 3 — Triggering an Error-Based SQL Injection

Using the PortSwigger SQL Injection cheat sheet, the following payload was crafted:

```sql
TrackingId=' AND CAST((SELECT password FROM users LIMIT 1) AS bool)--;
```

After modifying and testing the payload, the application generated a visible database error containing the administrator password.

![Visible Error Injection](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Visible%20Error-Based%20SQL%20Injection/screenshots/lab11(2).png?raw=true)

**Caption:** Extracting the administrator password through a visible database error.

---

## Step 4 — Logging into the Administrator Account

The extracted administrator credentials were used on the login page.

![Administrator Login](screenshots/error-based-sqli-4.png)

**Caption:** Attempting administrator login using extracted credentials.

---

## Step 5 — Successful Administrator Access

The administrator login succeeded successfully.

The lab was solved successfully.

![Administrator Access](screenshots/error-based-sqli-5.png)

**Caption:** Successful administrator login after extracting credentials via Error-Based SQL Injection.

---

# Why It Works

The application directly inserted user-controlled cookie data into backend SQL queries without proper sanitization.

The injected payload:

```sql
CAST((SELECT password FROM users LIMIT 1) AS bool)
```

attempted to:
1. retrieve a password from the database
2. convert that password into a boolean value

Because a password string cannot be converted into a boolean type properly, the database generated an error.

The error message accidentally exposed the queried password value, allowing the attacker to retrieve sensitive credentials directly.

---

# Understanding the LIMIT Keyword

The following part of the payload is important:

```sql
LIMIT 1
```

## Simple Meaning of LIMIT

`LIMIT` tells the database:

> "Only return a specific number of rows."

Example:

```sql
SELECT password FROM users LIMIT 1
```

means:

> "Give me only the first password from the users table."

---

## Why LIMIT Was Important in This Lab

Without `LIMIT 1`, the query might return:
- multiple passwords
- multiple rows

Example:

```sql
SELECT password FROM users
```

If multiple rows are returned:
- the query may fail
- the CAST function may not work correctly
- the error message may become unusable

Using:

```sql
LIMIT 1
```

ensures that:
- only one password is returned
- the query stays valid
- the database generates a cleaner error message

This helped the attacker extract a single administrator password successfully.

---

# Security Risk

This vulnerability can result in:
- credential disclosure
- database enumeration
- sensitive data leakage
- unauthorized administrator access
- complete database compromise

Error messages should never expose internal database information to users.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Disable verbose database error messages in production environments.
- Apply strict input validation.
- Restrict database permissions using least privilege principles.

---

# Learning Section

## What I Learned

- Error-based SQL Injection leaks sensitive information through database error messages.
- The `CAST()` function can intentionally trigger database conversion errors.
- `LIMIT` restricts query results to a specific number of rows.
- SQL Injection vulnerabilities can expose backend database values directly inside errors.
- Verbose error handling significantly increases application risk.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy
- PortSwigger SQL Injection Cheat Sheet

---

# Conclusion

This lab demonstrated a Visible Error-Based SQL Injection vulnerability through the `TrackingId` cookie.

By triggering a database type conversion error using the `CAST()` function, sensitive administrator credentials were exposed directly inside the application's error response, ultimately allowing unauthorized administrator access.
