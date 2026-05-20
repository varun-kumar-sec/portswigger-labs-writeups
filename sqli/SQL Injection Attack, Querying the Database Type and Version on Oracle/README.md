# SQL Injection Attack, Querying the Database Type and Version on Oracle

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | SQL injection attack, querying the database type and version on Oracle |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a UNION-based SQL Injection vulnerability used to identify the database type and retrieve the database version information.

The application filtered products using the following endpoint:

```text
/filter?category=gifts
```

The attacker first determined the number of columns returned by the backend query using the `ORDER BY` technique.

After identifying that the query returned **2 columns**, a UNION-based SQL Injection payload specific to Oracle databases was used to retrieve version information from the Oracle system table:

```sql
v$version
```

The database version banner was then displayed directly in the application's response.

---

# Vulnerability Type

- SQL Injection (SQLi)
- UNION-Based SQL Injection
- Database Enumeration
- Information Disclosure

---

# Impact

An attacker can retrieve backend database information such as:
- database type
- database version
- database vendor details

This information helps attackers:
- identify database-specific payloads
- craft advanced SQL Injection attacks
- exploit database-specific vulnerabilities
- improve attack accuracy

Database fingerprinting is an important step during SQL Injection exploitation.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Gifts** category.
3. Capture the following request using Burp Suite:

```text
/filter?category=gifts
```

4. Use the `ORDER BY` technique to determine the number of columns.

Example payload:

```sql
/filter?category=' ORDER BY 2--
```

5. Observe that the backend query contains **2 columns**.
6. Using the PortSwigger SQL Injection cheat sheet, craft the following payload:

```sql
/filter?category=' UNION SELECT banner,null FROM v$version--
```

7. Send the request.
8. Observe that the Oracle database version information is displayed in the application's response.
9. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal e-commerce website with product filtering functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20Attack,%20Querying%20the%20Database%20Type%20and%20Version%20on%20Oracle/screenshots/lab7.png?raw=true)

**Caption:** Initial application interface containing the Gifts category filter.

---

## Step 2 — Capturing the Product Filter Request

The request generated after selecting the **Gifts** category was intercepted using Burp Suite.

```text
/filter?category=gifts
```

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20Attack,%20Querying%20the%20Database%20Type%20and%20Version%20on%20Oracle/screenshots/lab7(1).png?raw=true)

**Caption:** Captured request using the category filter parameter.

---

## Step 3 — Determining the Number of Columns

The `ORDER BY` technique was used to identify the number of columns returned by the backend query.

Example payload:

```sql
/filter?category=' ORDER BY 2--
```

The query successfully returned data using 2 columns.

![ORDER BY Enumeration](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20Attack,%20Querying%20the%20Database%20Type%20and%20Version%20on%20Oracle/screenshots/lab7(2).png?raw=true)

**Caption:** Using the `ORDER BY` technique to determine the number of backend query columns.

---

## Step 4 — Retrieving Oracle Database Version Information

Using the PortSwigger SQL Injection cheat sheet, the following payload was crafted:

```sql
/filter?category=' UNION SELECT banner,null FROM v$version--
```

This payload queried Oracle's internal `v$version` table and displayed database version details inside the application's response.

The lab was solved successfully.

![Oracle Version Disclosure](screenshots/sqli-oracle-version-4.png)

**Caption:** Successfully retrieving Oracle database version information using SQL Injection.

---

# Why It Works

The application directly inserted user-controlled input into a SQL query without proper sanitization.

The attacker used:
- `ORDER BY` queries to identify the number of columns
- a UNION-based SQL Injection payload to retrieve data from Oracle system tables

The payload:

```sql
UNION SELECT banner,null FROM v$version--
```

worked because:
- the original query contained 2 columns
- the injected query also returned 2 columns
- the `banner` column contained readable string data
- Oracle exposes version information through the `v$version` system table

This allowed the attacker to fingerprint the backend database successfully.

---

# Security Risk

This vulnerability can result in:
- database fingerprinting
- information disclosure
- advanced attack preparation
- database-specific exploitation
- increased attack success rate

Information disclosure through SQL Injection significantly helps attackers during exploitation.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Apply strict input validation.
- Restrict database permissions using least privilege principles.
- Avoid exposing unnecessary database error messages or system information.

---

# Learning Section

## What I Learned

- SQL Injection can be used to fingerprint backend databases.
- Different databases use different system tables and syntax.
- Oracle stores version information inside the `v$version` table.
- UNION-based SQL Injection can retrieve system-level database information.
- Database version disclosure helps attackers craft targeted payloads.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy
- PortSwigger SQL Injection Cheat Sheet

---

# Conclusion

This lab demonstrated a UNION-based SQL Injection vulnerability that allowed retrieval of Oracle database version information.

By identifying the number of backend query columns and querying Oracle's internal `v$version` table, sensitive database fingerprinting information was successfully exposed through the application's response.
