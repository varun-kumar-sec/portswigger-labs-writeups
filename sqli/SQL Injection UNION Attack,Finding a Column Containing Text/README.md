# SQL Injection UNION Attack, Finding a Column Containing Text

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | SQL injection UNION attack, finding a column containing text |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a UNION-based SQL Injection vulnerability where the attacker identifies which database column can display string data in the application's response.

The application filtered products using the following endpoint:

```text
/filter?category=lifestyle
```

To perform a successful UNION-based SQL Injection attack, the attacker first determined the number of columns returned by the query using the `ORDER BY` technique.

After identifying that the query returned **3 columns**, a `UNION SELECT` statement was used to determine which column accepted string data.

The goal of the lab was to display the provided string:

```text
3GCj9s
```

inside the application's response.

---

# Vulnerability Type

- SQL Injection (SQLi)
- UNION-Based SQL Injection
- Database Query Manipulation

---

# Impact

An attacker can manipulate database queries and identify columns capable of displaying sensitive data.

This may lead to:
- database enumeration
- extraction of usernames and passwords
- sensitive data disclosure
- authentication bypass
- complete database compromise

Identifying string-compatible columns is an important step in advanced UNION-based SQL Injection attacks.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Lifestyle** category.
3. Capture the following request using Burp Suite:

```text
/filter?category=lifestyle
```

4. Use the `ORDER BY` technique to determine the number of columns.

Example:

```sql
ORDER BY 1--
ORDER BY 2--
ORDER BY 3--
ORDER BY 4--
```

5. Observe that:
   - valid column numbers return normal responses
   - invalid column numbers return server errors

6. In this lab:
   - `ORDER BY 1` → valid
   - `ORDER BY 2` → valid
   - `ORDER BY 3` → valid
   - `ORDER BY 4` → error

7. This confirmed that the query contained **3 columns**.
8. Validate the column count using:

```sql
/filter?category=' UNION SELECT null,null,null--
```

9. The lab required displaying the following text in the response:

```text
3GCj9s
```

10. Inject the following payload:

```sql
/filter?category=' UNION SELECT null,('3GCj9s'),null--
```

11. Observe that the text appears in the application's response.
12. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal e-commerce website containing product filtering functionality.

![Main Website](screenshots/sqli-union-text-1.png)

**Caption:** Initial application interface containing the Lifestyle category filter.

---

## Step 2 — Determining the Number of Columns

The request generated after selecting the **Lifestyle** category was captured using Burp Suite.

```text
/filter?category=lifestyle
```

The `ORDER BY` technique was used to determine the number of columns returned by the query.

In this case, the query contained **3 columns**.

![ORDER BY Enumeration](screenshots/sqli-union-text-2.png)

**Caption:** Using the `ORDER BY` technique to identify the number of columns.

---

## Step 3 — Validating the Column Count

A UNION query containing three `NULL` values was injected:

```sql
/filter?category=' UNION SELECT null,null,null--
```

This validated that the injected query matched the original query structure.

![UNION Validation](screenshots/sqli-union-text-3.png)

**Caption:** Validating the column count using a `UNION SELECT` query.

---

## Step 4 — Identifying the String-Compatible Column

The goal of the lab was to display the following text in the application's response:

```text
3GCj9s
```

The following payload was used:

```sql
/filter?category=' UNION SELECT null,'3GCj9s',null--
```

The text appeared successfully in the application's response, confirming that the second column accepted string data.

The lab was solved successfully.

![String Injection Successful](screenshots/sqli-union-text-4.png)

**Caption:** Successfully identifying the column capable of displaying string data.

---

# Why It Works

The application directly inserted user-controlled input into a SQL query without proper sanitization.

The attacker used:
- `ORDER BY` queries to identify the number of columns
- `UNION SELECT` statements to inject custom data into the query results

The payload:

```sql
UNION SELECT null,'3GCj9s',null--
```

worked because:
- the query contained exactly 3 columns
- the second column supported string data
- the application displayed that column in the HTTP response

This allowed attacker-controlled content to appear directly inside the webpage.

---

# Security Risk

This vulnerability can result in:
- sensitive data disclosure
- database enumeration
- extraction of credentials
- authentication bypass
- complete database compromise

UNION-based SQL Injection can expose large amounts of backend database information.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Apply strict input validation.
- Use ORM frameworks when possible.
- Restrict database permissions using least privilege principles.

---

# Learning Section

## What I Learned

- UNION attacks require matching the number of columns in the original query.
- The `ORDER BY` technique can determine column count.
- `NULL` values are useful for testing UNION compatibility.
- String-compatible columns can display attacker-controlled data.
- UNION-based SQL Injection is commonly used for database enumeration and data extraction.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a UNION-based SQL Injection vulnerability caused by insecure handling of user-controlled input.

By identifying the number of columns and determining which column accepted string data, attacker-controlled content was successfully injected into the application's response using a crafted `UNION SELECT` statement.
