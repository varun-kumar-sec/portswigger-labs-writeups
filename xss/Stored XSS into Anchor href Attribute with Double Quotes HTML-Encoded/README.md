# Stored XSS into Anchor href Attribute with Double Quotes HTML-Encoded

## 📌 Lab Overview

This lab demonstrates a **Stored Cross-Site Scripting (Stored XSS)** vulnerability inside an HTML anchor (`href`) attribute.

Unlike normal Stored XSS:
- the vulnerability existed inside a hyperlink attribute
- angle bracket injection was unnecessary
- JavaScript execution occurred through the `javascript:` protocol

The application stored attacker-controlled input inside a clickable link without properly validating the URL scheme.

---

# 🔍 What is an href Attribute?

The `href` attribute is used inside anchor (`<a>`) tags to define where a link should navigate.

Example:

```html
<a href="https://google.com">Visit</a>
```

When users click the link:
- browser opens the specified destination

---

# 🔍 What is `javascript:`?

Normally, links use protocols such as:
- `http://`
- `https://`

However, browsers also support:

```javascript
javascript:
```

Example:

```html
<a href="javascript:alert(1)">Click Me</a>
```

When the link is clicked:
- browser executes JavaScript instead of opening a webpage

---

# 🔍 Understanding the Payload

The payload used in this lab was:

```javascript
javascript:alert(1)
```

---

# 🔍 Payload Breakdown

## `javascript:`

This tells the browser:
- treat the following content as JavaScript code
- execute it directly instead of navigating to a website

---

## `alert(1)`

This executes JavaScript and displays a popup box containing:

```text
1
```

This is commonly used in XSS labs to confirm successful code execution.

---

# 🎯 Objective

The goal of this lab was to inject a malicious JavaScript URL into the website field and trigger JavaScript execution through the generated hyperlink.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20XSS%20into%20Anchor%20href%20Attribute%20with%20Double%20Quotes%20HTML-Encoded/screenshots/lab8(1).png?raw=true)

The application initially displayed a normal webpage containing a **View Post** functionality.

At this stage:
- no payload was injected
- application behaved normally
- blog posts were accessible

---

## Screenshot 2 — Injecting the Payload

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20XSS%20into%20Anchor%20href%20Attribute%20with%20Double%20Quotes%20HTML-Encoded/screenshots/lab8(2).png?raw=true)

After opening a blog post:
- a **Leave a Comment** section became visible

The lab description hinted that:
- the vulnerability existed inside the `href` attribute
- the most relevant input field was the **Website** field

The following values were submitted:

### Comment

```text
hello
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

```javascript
javascript:alert(1)
```

---

# 🔍 Why the Website Field Was Vulnerable

Applications often generate profile links like:

```html
<a href="USER_WEBSITE">Website</a>
```

If the application blindly trusts user input:

```html
<a href="javascript:alert(1)">Website</a>
```

then clicking the link executes JavaScript directly inside the browser.

---

## Screenshot 3 — Successful XSS Execution

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20XSS%20into%20Anchor%20href%20Attribute%20with%20Double%20Quotes%20HTML-Encoded/screenshots/lab8(3).png?raw=true)

After submitting the comment:
- the malicious website value was stored successfully
- the application generated a vulnerable hyperlink
- JavaScript execution occurred successfully
- the XSS vulnerability was confirmed

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- user-controlled input was inserted directly into an `href` attribute
- the application failed to validate dangerous protocols
- `javascript:` URLs were allowed

Example vulnerable behavior:

```html
<a href="javascript:alert(1)">Website</a>
```

Browsers interpret:
- `javascript:` as executable code
- not as a normal webpage URL

---

# 💥 Impact of Stored XSS

An attacker can potentially:
- steal session cookies
- hijack user accounts
- redirect users to malicious pages
- perform phishing attacks
- execute arbitrary JavaScript
- manipulate webpage content

Stored XSS is especially dangerous because:
- the malicious payload remains permanently stored
- multiple users can become victims automatically

---

# 🛡 Mitigation

To prevent href-based XSS:
- validate URL schemes properly
- allow only trusted protocols such as:
  - `http://`
  - `https://`
- block dangerous protocols:
  - `javascript:`
  - `data:`
- sanitize user-controlled input
- implement Content Security Policy (CSP)

Safe validation example:

```javascript
if(url.startsWith('https://'))
```

---

# 🧠 Skills Learned

- Understanding href attribute injection
- JavaScript protocol exploitation
- Stored XSS techniques
- URL scheme abuse
- Browser hyperlink behavior
- Attribute-based JavaScript execution

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how unsafe handling of user-controlled URLs inside anchor (`href`) attributes can lead to Stored Cross-Site Scripting vulnerabilities.

By injecting a `javascript:` payload into the website field, it was possible to execute arbitrary JavaScript through a generated hyperlink.

Through this lab, I learned:
- how `javascript:` URLs work
- how browsers execute JavaScript inside hyperlinks
- how attribute-based XSS vulnerabilities occur
- why URL validation is important
- how Stored XSS can persist across users

The lab was successfully exploited by injecting a malicious `javascript:` URL inside the website field, resulting in JavaScript execution through the generated hyperlink.
