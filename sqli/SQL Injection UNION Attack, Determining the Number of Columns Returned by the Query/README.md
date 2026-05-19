# SQL Injection UNION Attack, Determining the Number of Columns Returned by the Query

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | SQL injection UNION attack, determining the number of columns returned by the query |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a UNION-based SQL Injection vulnerability where the attacker determines the number of columns returned by the original database query.

The application filtered products using the following endpoint:

```text
/filter?category=Gifts
```

To perform a successful UNION attack, the number of columns in the original query must match the number of columns supplied in the injected `UNION SELECT` statement.

The `ORDER BY` technique was used to identify the total number of columns returned by the query.

---

# Vulnerability Type

- SQL Injection (SQLi)
- UNION-Based SQL Injection
- Database Query Manipulation

---

# Impact

An attacker can manipulate SQL queries to retrieve unauthorized data from the backend database.

This may lead to:
- database enumeration
- sensitive data exposure
- extraction of credentials
- authentication bypass
- complete database compromise

UNION-based SQL Injection allows attackers to combine malicious queries with legitimate database queries.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Gifts** category.
3. Capture the following request using Burp Suite:

```text
/filter?category=Gifts
```

4. Use the `ORDER BY` technique to determine the number of columns.

Example payload:

```sql
/filter?category=' ORDER BY 1--
```

5. Increase the number sequentially:

```sql
ORDER BY 2--
ORDER BY 3--
ORDER BY 4--
```

6. Observe the server responses:
   - Valid column numbers return:

```http
200 OK
```

   - Invalid column numbers return:

```http
500 Internal Server Error
```

7. In this lab:
   - `ORDER BY 1` → valid
   - `ORDER BY 2` → valid
   - `ORDER BY 3` → valid
   - `ORDER BY 4` → error

8. This indicates that the query contains **3 columns**.
9. Perform the UNION attack using:

```sql
/filter?category=' UNION SELECT null,null,null--
```

10. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal e-commerce website with product filtering functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Determining%20the%20Number%20of%20Columns%20Returned%20by%20the%20Query/screenshots/lab3.png?raw=true)

**Caption:** Initial application interface containing the Gifts category filter.

---

## Step 2 — Capturing the Request

The request generated after selecting the **Gifts** category was intercepted using Burp Suite.

```text
/filter?category=Gifts
```

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Determining%20the%20Number%20of%20Columns%20Returned%20by%20the%20Query/screenshots/lab3(1).png?raw=true)

**Caption:** Captured product filter request using the category parameter.

---

## Step 3 — Determining the Number of Columns

The `ORDER BY` technique was used to determine the number of columns returned by the query.

Example payload:

```sql
/filter?category=' ORDER BY 1--
```

The server behavior was:
- valid column number → `200 OK`
- invalid column number → `500 Internal Server Error`

After testing:
- `ORDER BY 1` → success
- `ORDER BY 2` → success
- `ORDER BY 3` → success
- `ORDER BY 4` → error

This confirmed that the query contained **3 columns**.

![ORDER BY Technique](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Determining%20the%20Number%20of%20Columns%20Returned%20by%20the%20Query/screenshots/lab3(2).png?raw=true)

**Caption:** Using the `ORDER BY` technique to determine the number of columns.

---

## Step 4 — Performing the UNION Attack

A UNION query containing three `NULL` values was injected:

```sql
/filter?category=' UNION SELECT null,null,null--
```

The number of `NULL` values matched the number of columns in the original query.

The lab was solved successfully.

![UNION SELECT Payload](screenshots/sqli-union-columns-4.png)

**Caption:** Successful UNION-based SQL Injection after matching the number of columns.

---

# Why It Works

The application directly inserted user-controlled input into a SQL query without proper sanitization.

The original backend query likely resembled:

```sql
SELECT name, description, price 
FROM products 
WHERE category = 'Gifts'
```

The attacker injected:

```sql
UNION SELECT null,null,null--
```

The `UNION` operator combines results from two separate queries.

For a successful UNION attack:
- both queries must contain the same number of columns
- compatible data types must be used

The `ORDER BY` technique helped identify the correct number of columns before performing the UNION attack.

---

# Security Risk

This vulnerability can result in:
- database enumeration
- sensitive data extraction
- credential disclosure
- authentication bypass
- complete database compromise

UNION-based SQL Injection is one of the most dangerous forms of SQL Injection.

---

# Recommended Mitigation

- Use parameterized queries (prepared statements).
- Never concatenate user input into SQL queries.
- Apply strict input validation.
- Use ORM frameworks when possible.
- Restrict database account privileges.

---

# Learning Section

## What I Learned

- UNION attacks require matching the number of columns in the original query.
- The `ORDER BY` technique helps identify column count.
- `NULL` values are commonly used during UNION testing because they are compatible with most data types.
- SQL Injection can manipulate backend database logic.
- Improper query construction creates critical security vulnerabilities.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a UNION-based SQL Injection vulnerability caused by insecure handling of user input inside SQL queries.

By using the `ORDER BY` technique, the correct number of columns was identified. A successful `UNION SELECT` query was then crafted using matching column counts, allowing manipulation of the backend database query and solving the lab.
