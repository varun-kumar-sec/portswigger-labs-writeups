# Cross-Site Scripting (XSS) Labs

This repository contains my hands-on practice and writeups for **Cross-Site Scripting (XSS)** vulnerabilities from the **PortSwigger Web Security Academy**.

The goal of this repository is to document:
- XSS exploitation techniques
- Payload crafting
- Filter bypasses
- DOM-based vulnerabilities
- Browser-side injection attacks
- Real-world attack impact
- Secure mitigation methods

All labs were solved manually using Burp Suite and browser developer tools.

---

# What is XSS?

Cross-Site Scripting (XSS) is a web vulnerability that allows attackers to inject malicious JavaScript into a trusted website.

When another user visits the vulnerable page, the injected JavaScript executes inside their browser.

XSS vulnerabilities can lead to:
- session hijacking
- cookie theft
- account takeover
- phishing attacks
- website defacement
- keylogging
- malicious redirects
- unauthorized actions

---

# Types of XSS Covered

## Reflected XSS
Payload is reflected immediately in the server response.

Example:
```html
<script>alert(1)</script>
```

---

## Stored XSS
Payload gets permanently stored in the application database and executes whenever users view the affected page.

---

## DOM-Based XSS
The vulnerability exists inside client-side JavaScript instead of server-side code.

Common dangerous sinks:
- `innerHTML`
- `document.write`
- `eval()`

---

# Repository Structure

```text
portswigger-xss-labs/
│
├── reflected-xss/
├── stored-xss/
├── dom-xss/
├── advanced-xss/
└── README.md
```

---

# Topics Covered

- Reflected XSS
- Stored XSS
- DOM XSS
- Attribute Injection
- JavaScript Context Injection
- HTML Context Injection
- Event Handler Injection
- Cookie Theft
- CSP Bypass
- AngularJS Sandbox Escape
- Filter Bypass Techniques
- Web Message Exploitation

---

# Tools Used

- Burp Suite
- Burp Repeater
- Burp Intruder
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Skills Learned

- Understanding XSS contexts
- Breaking out of HTML attributes
- Injecting JavaScript payloads
- Identifying DOM sinks and sources
- Bypassing filters and encodings
- Exploiting browser-side vulnerabilities
- Understanding Content Security Policy (CSP)
- Payload crafting for different contexts

---

# Example Payloads

## Basic Alert Payload

```html
<script>alert(1)</script>
```

---

## Image Event Handler Payload

```html
<img src=x onerror=alert(1)>
```

---

## SVG Payload

```html
<svg onload=alert(1)>
```

---

## Attribute Injection Payload

```html
" onmouseover="alert(1)
```

---

# Security Impact

XSS vulnerabilities can result in:
- account takeover
- session hijacking
- credential theft
- phishing attacks
- malicious redirects
- unauthorized actions
- sensitive information disclosure

---

# Mitigation

- Proper output encoding
- Input validation
- Context-aware escaping
- Content Security Policy (CSP)
- HttpOnly cookies
- Avoid dangerous DOM sinks
- Use secure frameworks and templating engines

---

# Learning Resources

- PortSwigger Web Security Academy
- OWASP XSS Cheat Sheet
- Mozilla Developer Documentation (MDN)

---

# Disclaimer

This repository is created strictly for:
- educational purposes
- ethical hacking practice
- security research
- vulnerability learning

All labs were performed in legal training environments provided by PortSwigger.
