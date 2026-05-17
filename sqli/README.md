# SQL Injection Labs

## 📌 Overview

This section contains SQL Injection (SQLi) labs from PortSwigger Web Security Academy.

SQL Injection is one of the most critical web application vulnerabilities and occurs when user-controlled input is inserted directly into SQL queries without proper sanitization or parameterized handling.

These labs demonstrate how attackers can manipulate database queries to:
- retrieve hidden data
- bypass authentication
- enumerate databases
- extract sensitive information
- escalate privileges
- compromise backend systems

---

# 🛠 Skills Practiced

- SQL query manipulation
- Authentication bypass
- UNION-based SQL Injection
- Database enumeration
- Blind SQL Injection
- Time-based SQL Injection
- Payload crafting
- HTTP request manipulation
- Burp Suite testing methodology

---

# 📂 Labs Completed

## Apprentice Labs

- [x] SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- [x] SQL injection vulnerability allowing login bypass
- [ ] SQL injection UNION attack, determining the number of columns returned by the query
- [ ] SQL injection UNION attack, finding a column containing text
- [ ] SQL injection UNION attack, retrieving data from other tables
- [ ] SQL injection UNION attack, retrieving multiple values in a single column

---

## Practitioner Labs

- [ ] SQL injection attack, querying the database type and version
- [ ] SQL injection attack, listing the database contents on non-Oracle databases
- [ ] Blind SQL injection with conditional responses
- [ ] Blind SQL injection with time delays and information retrieval
- [ ] Visible error-based SQL injection
- [ ] Blind SQL injection with out-of-band interaction
- [ ] Blind SQL injection with out-of-band data exfiltration

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# 📖 Learning Goals

Through these labs, I practiced:
- understanding SQL query behavior
- exploiting vulnerable input fields
- manipulating backend database logic
- identifying insecure query construction
- performing manual SQL Injection testing
- understanding the impact of insecure database interactions

---

# ⚠️ Disclaimer

These labs were performed in a legal training environment provided by PortSwigger Web Security Academy for educational and ethical learning purposes only.
