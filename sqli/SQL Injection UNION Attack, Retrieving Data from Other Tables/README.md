# SQL Injection UNION Attack, Retrieving Data from Other Tables

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | SQL injection UNION attack, retrieving data from other tables |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a UNION-based SQL Injection vulnerability where sensitive data was extracted from another database table.

The application filtered products using the following endpoint:

```text
/filter?category=gifts
```

The attacker first determined the number of columns returned by the original query using the `ORDER BY` technique.

After identifying that the query returned **2 columns**, a crafted `UNION SELECT` statement was used to retrieve usernames and passwords from the `users` table.

The lab description already provided:
- the table name → `users`
- the column names → `username` and `password`

Using this information, the administrator credentials were extracted and used to authenticate successfully.

---

# Vulnerability Type

- SQL Injection (SQLi)
- UNION-Based SQL Injection
- Sensitive Data Extraction
- Database Enumeration

---

# Impact

An attacker can retrieve sensitive data directly from backend database tables.

This may lead to:
- credential disclosure
- unauthorized account access
- administrator compromise
- sensitive data exposure
- complete database compromise

UNION-based SQL Injection is highly dangerous because it allows direct extraction of backend database information.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Gifts** category.
3. Capture the following request using Burp Suite:

```text
/filter?category=gifts
```

4. Use the `ORDER BY` technique to determine the number of columns.

Example:

```sql
/filter?category=' ORDER BY 1--
/filter?category=' ORDER BY 2--
/filter?category=' ORDER BY 3--
```

5. Observe the server responses:
   - valid column numbers return normal responses
   - invalid column numbers return server errors

6. In this lab:
   - `ORDER BY 1` → valid
   - `ORDER BY 2` → valid
   - `ORDER BY 3` → error

7. This confirmed that the query contained **2 columns**.
8. Inject the following UNION query:

```sql
/filter?category=' UNION SELECT username,password FROM users--
```

9. Observe that usernames and passwords are displayed in the application's response.
10. Copy the administrator credentials.
11. Navigate to the login page.
12. Login using the extracted administrator credentials.
13. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal e-commerce website with product filtering functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Data%20from%20Other%20Tables/screenshots/lab5.png?raw=true)

**Caption:** Initial application interface containing the Gifts category filter.

---

## Step 2 — Capturing the Product Filter Request

The request generated after selecting the **Gifts** category was intercepted using Burp Suite.

```text
/filter?category=gifts
```

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Data%20from%20Other%20Tables/screenshots/lab5(1).png?raw=true)

**Caption:** Captured request using the category filter parameter.

---

## Step 3 — Determining the Number of Columns

The `ORDER BY` technique was used to identify the number of columns returned by the original query.

Example payload:

```sql
/filter?category=' ORDER BY 2--
```

The application returned an error after testing column number `3`, confirming that the query contained **2 columns**.

![ORDER BY Enumeration](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Data%20from%20Other%20Tables/screenshots/lab5(2).png?raw=true)

**Caption:** Using the `ORDER BY` technique to determine the number of backend query columns.

---

## Step 4 — Retrieving Data from the Users Table

The following UNION-based SQL Injection payload was used:

```sql
/filter?category=' UNION SELECT username,password FROM users--
```

The lab description already provided:
- table name → `users`
- columns → `username`, `password`

After executing the payload, usernames and passwords became visible in the application's response.

The administrator password was successfully retrieved.

![Credential Extraction](screenshots/sqli-union-data-4.png)

**Caption:** Extracting usernames and passwords from the backend users table.

---

## Step 5 — Logging into the Administrator Account

The extracted administrator credentials were used on the login page.

![Administrator Login](screenshots/sqli-union-data-5.png)

**Caption:** Attempting login using the extracted administrator credentials.

---

## Step 6 — Successful Administrator Access

The login succeeded successfully, granting access to the administrator account.

The lab was solved successfully.

![Administrator Access](screenshots/sqli-union-data-6.png)

**Caption:** Successful administrator login using credentials extracted via SQL Injection.

---

# Why It Works

The application directly inserted user-controlled input into a SQL query without proper sanitization.

The original backend query likely resembled:

```sql
SELECT name, description 
FROM products 
WHERE category = 'gifts'
```

The attacker injected:

```sql
UNION SELECT username,password FROM users--
```

The `UNION` operator combines results from multiple SQL queries.

Because:
- the original query contained 2 columns
- the injected query also returned 2 columns
- compatible data types were used

the database successfully returned data from the `users` table inside the application's response.

This exposed sensitive credentials belonging to application users.

---

# Security Risk

This vulnerability can result in:
- credential disclosure
- unauthorized administrator access
- sensitive data exposure
- privilege escalation
- complete database compromise

SQL Injection vulnerabilities allowing data extraction are considered critical.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Apply strict input validation.
- Use ORM frameworks when possible.
- Restrict database account permissions using least privilege principles.

---

# Learning Section

## What I Learned

- UNION attacks can retrieve data from completely different database tables.
- Matching column counts is essential for successful UNION-based SQL Injection.
- Sensitive database information such as usernames and passwords can be extracted through SQL Injection.
- `ORDER BY` queries help identify backend query structure.
- Improper query construction can expose critical backend data.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a UNION-based SQL Injection vulnerability that allowed extraction of sensitive data from another database table.

By identifying the correct number of columns and crafting a malicious `UNION SELECT` query, usernames and passwords were extracted from the backend database, ultimately allowing unauthorized administrator access.
