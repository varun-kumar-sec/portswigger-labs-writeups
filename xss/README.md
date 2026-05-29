# Cross-Site Scripting (XSS) Labs

---

# 📌 Overview

This section contains Cross-Site Scripting (XSS) labs from PortSwigger Web Security Academy.

Cross-Site Scripting (XSS) is a web application vulnerability that allows attackers to inject malicious JavaScript into trusted web applications.

These labs demonstrate how attackers can:
- execute malicious JavaScript
- steal session cookies
- perform account takeover
- manipulate webpage content
- bypass browser security mechanisms
- exploit client-side vulnerabilities

---

# 🛠 Skills Practiced

- Reflected XSS exploitation
- Stored XSS exploitation
- DOM-based XSS testing
- JavaScript payload crafting
- HTML context injection
- Attribute context injection
- Event handler exploitation
- DOM sink identification
- Browser-side vulnerability testing
- Filter bypass techniques
- CSP bypass basics
- Burp Suite testing methodology

---

# 📂 Labs Completed

## Apprentice Labs

☑ Reflected XSS into HTML context with nothing encoded

☑ Reflected XSS into attribute with angle brackets HTML-encoded

☑ Stored XSS into HTML context with nothing encoded

☑ DOM XSS in document.write sink using source location.search

☑ DOM XSS in innerHTML sink using source location.search

---

## Practitioner Labs

☑ Reflected XSS into JavaScript string with single quote and backslash escaped

☑ Reflected XSS into HTML context with most tags and attributes blocked

☑ Stored XSS into anchor href attribute with double quotes HTML-encoded

☑ Stored DOM XSS

☑ DOM XSS using web messages

☑ Exploiting cross-site scripting to steal cookies

☑ Exploiting XSS to perform CSRF

☑ Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped


---

## Expert Labs

☑ Reflected XSS protected by CSP, with CSP bypass

☑ Reflected XSS with AngularJS sandbox escape

☑ Reflected XSS with event handlers and href attributes blocked
---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Burp Intruder
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# 📖 Learning Goals

Through these labs, I practiced:

- understanding browser parsing behavior
- identifying vulnerable client-side code
- crafting context-specific payloads
- bypassing filters and encodings
- exploiting DOM-based vulnerabilities
- understanding JavaScript execution contexts
- identifying dangerous DOM sinks and sources
- understanding the impact of insecure frontend logic

---

# ⚠️ Disclaimer

These labs were performed in a legal training environment provided by PortSwigger Web Security Academy for educational and ethical learning purposes only.
