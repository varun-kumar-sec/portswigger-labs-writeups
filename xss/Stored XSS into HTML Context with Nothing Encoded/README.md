# Stored XSS into HTML Context with Nothing Encoded

## 📌 Lab Overview

This lab demonstrates a **Stored Cross-Site Scripting (Stored XSS)** vulnerability.

### What is Stored XSS?

Stored XSS occurs when malicious user input is:
- permanently stored by the application
- saved inside a database, comment section, profile field, or message system
- later displayed to other users without proper sanitization

When victims visit the affected page, the malicious JavaScript executes automatically inside their browser.

Unlike Reflected XSS:
- the payload does not require a specially crafted link every time
- the malicious code stays stored inside the application

Stored XSS is generally more dangerous because it can affect multiple users automatically.

---

# 🎯 Objective

The goal of this lab was to inject a malicious JavaScript payload into the comment section and execute it when the blog post loaded.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-xss1.png)

The application initially displayed a normal webpage containing a **View Post** functionality.

At this stage:
- no payload was injected
- application behaved normally
- blog posts could be accessed

---

## Screenshot 2 — Inspecting the Comment Section

![Screenshot 2](screenshot-xss2.png)

After opening a blog post:
- a comments section became visible
- multiple users had already posted comments

While inspecting the page source, it became visible that comments were directly inserted inside a paragraph tag:

```html
<p>User Comment</p>
```

This indicated that:
- user-controlled input was being rendered directly into HTML
- if malicious JavaScript was inserted between the paragraph tags, it could execute inside the browser

This became the injection point for Stored XSS.

---

## Screenshot 3 — Injecting the XSS Payload

![Screenshot 3](screenshot-xss3.png)

The following payload was submitted through the **Leave a Comment** section:

### Comment

```html
<script>alert('xss')</script>
```

### Name

```text
abcd
```

### Email

```text
abcd@gmail.com
```

### Website

```text
http://google.com
```

The payload was designed to test whether the application:
- sanitizes comments
- filters script tags
- safely renders user-controlled content

---

## Screenshot 4 — Comment Submission Response

![Screenshot 4](screenshot-xss4.png)

After submitting the comment:
- the application accepted the payload
- the comment was stored successfully
- a confirmation message appeared:

```text
Thank you for your comment! Your comment has been submitted.
```

A **Back to Blog** button was also provided.

This confirmed that:
- the application stored the malicious payload inside the backend database
- no sanitization or filtering occurred during submission

---

## Screenshot 5 — Successful Stored XSS Execution

![Screenshot 5](screenshot-xss5.png)

After clicking the **Back to Blog** button:
- the malicious comment loaded automatically
- the browser interpreted the `<script>` tag as JavaScript
- an XSS popup appeared

This confirmed:
- successful Stored XSS exploitation
- unsafe rendering of stored user input
- execution of attacker-controlled JavaScript

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The application stored user-controlled input and later rendered it directly into the webpage without:
- sanitization
- output encoding
- input validation

Example vulnerable behavior:

```html
<p><script>alert('xss')</script></p>
```

Because browsers trust script tags rendered inside HTML pages, the JavaScript executes automatically whenever users load the page.

---

# 💥 Impact of Stored XSS

An attacker can potentially:
- steal session cookies
- hijack user accounts
- perform phishing attacks
- redirect users to malicious websites
- manipulate webpage content
- execute actions on behalf of victims
- infect every user visiting the page

Stored XSS is especially dangerous because:
- the payload remains persistent
- multiple users can become victims automatically

---

# 🛡 Mitigation

To prevent Stored XSS:
- sanitize user-controlled input
- perform proper output encoding
- escape dangerous HTML characters
- implement Content Security Policy (CSP)
- avoid rendering raw HTML from users
- validate all input before storage

---

# 🧠 Skills Learned

- Understanding Stored XSS behavior
- Persistent JavaScript injection
- HTML context injection
- Browser-side JavaScript execution
- Identifying unsafe rendering behavior
- Inspecting HTML source for injection points
- Testing comment-based vulnerabilities

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improperly sanitized user comments can lead to Stored Cross-Site Scripting vulnerabilities.

By injecting a malicious JavaScript payload into the comment section, it was possible to permanently store attacker-controlled code inside the application and execute it automatically when the blog page loaded.

Through this lab, I learned:
- how Stored XSS works
- why persistent XSS is dangerous
- how browsers execute stored JavaScript
- how unsafe HTML rendering leads to code execution
- why output encoding and sanitization are critical for user-generated content

The lab was successfully exploited by injecting a script payload inside the comment section, resulting in automatic JavaScript execution when the page was revisited.
