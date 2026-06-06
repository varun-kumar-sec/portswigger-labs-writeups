# PortSwigger Labs Writeups

## 📌 Overview
This repository contains practical writeups and walkthroughs for labs completed from the PortSwigger Web Security Academy. The goal of this repository is to document hands-on experience in identifying, exploiting, and understanding common web application vulnerabilities using industry-standard tools and methodologies.

---

## 🎯 Objectives
- Practice real-world web application security testing
- Improve vulnerability assessment skills
- Gain hands-on experience with Burp Suite
- Understand OWASP Top 10 vulnerabilities
- Document lab methodologies and exploitation techniques

---

## 🛠 Tools Used
- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

## 📂 Lab Categories

## Access Control
- [x] Unprotected admin functionality
- [x] Unprotected admin functionality with unpredictable URL
- [x] User role controlled by request parameter
- [x] User role can be modified in user profile
- [x] User ID controlled by request parameter 
- [x] URL-based access control can be circumvented
- [x] Method-Based Access Control Can Be Circumvented

---

## Authentication
- [x] Username enumeration via different responses
- [x] Username enumeration via subtly different responses
- [x] Username enumeration via response timing
- [x] Username enumeration via account lock
- [x] Broken brute force protection, IP block
- [x] Broken brute-force protection, multiple credentials per request
- [x] Password brute-force via password change
- [x] 2FA simple bypass
- [x] 2FA bypass using a brute-force attack
- [x] Password reset broken logic
- [x] Password reset poisoning via middleware
- [x] Brute-forcing a stay-logged-in cookie
- [x] Offline password cracking

---

## Cross-Site Scripting (XSS)
- [x] Reflected XSS into HTML context with nothing encoded
- [x] Reflected XSS into attribute with angle brackets HTML-encoded
- [x] Stored XSS into HTML context with nothing encoded
- [x] DOM XSS in document.write sink using source location.search
- [x] DOM XSS in innerHTML sink using source location.search
- [x] Reflected XSS into JavaScript string with single quote and backslash escaped
- [x] Reflected XSS into HTML context with most tags and attributes blocked
- [x] Stored XSS into anchor href attribute with double quotes HTML-encoded
- [x] Stored DOM XSS
- [x] Reflected DOM XSS
- [x] Exploiting cross-site scripting to steal cookies
- [x] Exploiting XSS to perform CSRF
- [x] Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped
- [x] Reflected XSS with AngularJS sandbox escape
- [x] Reflected XSS protected by CSP, with CSP bypass
- [x] Reflected XSS with event handlers and href attributes blocked

---

## SQL Injection
- [x] SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
- [x] SQL injection vulnerability allowing login bypass
- [x] SQL injection UNION attack, determining the number of columns returned by the query
- [x] SQL injection UNION attack, finding a column containing text
- [x] SQL injection UNION attack, retrieving data from other tables
- [x] SQL injection UNION attack, retrieving multiple values in a single column
- [x] SQL injection attack, querying the database type and version on oracle
- [x] SQL injection attack, listing the database contents on non-Oracle databases
- [x] Blind SQL injection with conditional responses
- [x] Blind SQL injection with time delays and information retrieval
- [x] Visible error-based SQL injection
- [x] Blind SQL injection with out-of-band interaction
- [x] Blind SQL injection with out-of-band data exfiltration

---

## 📖 Disclaimer
This repository is created strictly for educational and ethical security learning purposes. All testing was performed within intentionally vulnerable lab environments provided by PortSwigger Web Security Academy.

---

## 🚀 Author
Varun Kumar Dabde
