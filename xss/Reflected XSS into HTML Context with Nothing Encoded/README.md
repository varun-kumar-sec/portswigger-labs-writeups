# Reflected XSS into HTML Context with Nothing Encoded

## 📌 Lab Overview

This lab demonstrates a basic **Reflected Cross-Site Scripting (XSS)** vulnerability.

### What is Reflected XSS?

Reflected XSS occurs when user-controlled input is immediately reflected back in the server response without proper sanitization or encoding.

Because the application directly includes user input inside the webpage, attackers can inject malicious JavaScript code that executes inside the victim's browser.

Unlike Stored XSS:
- the payload is **not stored permanently**
- it executes only when the malicious request is processed

---

# 🎯 Objective

The goal of this lab was to inject JavaScript code into the search functionality and execute it in the browser.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-xss1.png)

The lab initially provided a normal webpage containing a search functionality.

At this stage:
- no payload was injected
- application behaved normally
- search feature was available for testing

---

## Screenshot 2 — Injecting XSS Payload

![Screenshot 2](screenshot-xss2.png)

The following payload was inserted into the search bar:

```html
<script>alert('xss')</script>
```

### Payload Explanation

- `<script>` → starts JavaScript execution
- `alert('xss')` → creates a popup message
- `</script>` → closes the script tag

The purpose of this payload was to test whether the application properly sanitizes user input before reflecting it into the webpage.

---

## Screenshot 3 — Successful XSS Execution

![Screenshot 3](screenshot-xss3.png)

After clicking the search button:
- the payload was reflected directly into the webpage
- browser interpreted the injected script as valid JavaScript
- an alert popup appeared in the browser

This confirmed that:
- user input was not sanitized
- user input was inserted directly into HTML context
- the application was vulnerable to Reflected XSS

The lab was successfully solved after the popup executed.

---

# ⚠ Why This Vulnerability Happens

The application directly inserted user-controlled input into the HTML response without:
- output encoding
- sanitization
- input validation

Example vulnerable behavior:

```html
Search Results for: <script>alert('xss')</script>
```

Because browsers trust script tags inside HTML pages, the JavaScript gets executed automatically.

---

# 💥 Impact of Reflected XSS

An attacker can potentially:
- steal session cookies
- hijack user accounts
- redirect users to malicious websites
- perform phishing attacks
- execute actions on behalf of users
- manipulate webpage content

---

# 🛡 Mitigation

To prevent Reflected XSS:
- perform proper output encoding
- sanitize user-controlled input
- use Content Security Policy (CSP)
- avoid directly rendering user input into HTML
- use secure frontend templating frameworks

---

# 🧠 Skills Learned

- Understanding reflected XSS behavior
- HTML context injection
- JavaScript payload execution
- Browser-side code execution
- Identifying insecure input reflection
- Basic XSS testing methodology

---

# 🧰 Tools Used

- Burp Suite
- Firefox Browser
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how unsafe reflection of user input inside HTML pages can lead to Reflected Cross-Site Scripting vulnerabilities.

By injecting a simple JavaScript payload into the search functionality, it was possible to execute arbitrary code inside the browser because the application failed to sanitize user-controlled input.
Through this lab, I learned:
- how reflected XSS works
- how browsers execute injected JavaScript
- how HTML context injection occurs
- how attackers test for client-side vulnerabilities
- the importance of proper output encoding and sanitization

The lab was successfully exploited by triggering a JavaScript alert popup using an XSS payload.
