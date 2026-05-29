# Reflected XSS Protected by CSP, with CSP Bypass

## 📌 Lab Overview

This lab demonstrates how a **Content Security Policy (CSP)** can be bypassed to achieve successful Reflected Cross-Site Scripting (XSS).

Initially:
- the XSS payload was reflected correctly
- but JavaScript execution was blocked by the browser

The reason:
- the application implemented a CSP policy

However:
- the CSP configuration itself contained an injection point
- this allowed modification of the policy
- attacker-controlled directives weakened the protection
- inline JavaScript execution became possible

This lab focused on:
- understanding CSP
- CSP directives
- CSP bypass techniques
- modifying browser security policies

---

# 🔍 What is CSP?

CSP stands for:

```text
Content Security Policy
```

It is a browser security mechanism designed to:
- reduce XSS attacks
- control which resources browsers are allowed to load
- restrict JavaScript execution

The server sends CSP rules inside HTTP response headers.

Example:

```http
Content-Security-Policy: script-src 'self'
```

The browser then follows those security rules strictly.

---

# 🔍 Why CSP Exists

Even if:
- attackers inject JavaScript successfully

CSP can still block execution.

This acts as:
- an additional security layer against XSS

---

# 🔍 How CSP Works

The server defines rules such as:
- allowed JavaScript sources
- allowed CSS sources
- allowed image sources
- blocked resources
- reporting endpoints

The browser enforces these rules automatically.

---

# 🎯 Objective

The goal of this lab was to:
- identify the CSP protection
- discover a CSP injection point
- inject new CSP directives
- bypass the CSP restrictions
- execute JavaScript successfully

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Initial XSS Attempt

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(1).png?raw=true)

The application initially displayed a normal webpage with a search functionality.

A basic payload was entered:

```html
<script>alert(1)</script>
```

---

## Screenshot 2 — Payload Reflected but Not Executed

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(2).png?raw=true)

After searching:
- the payload appeared correctly inside the page source
- but JavaScript execution never occurred

This indicated:
- the payload was reflected successfully
- another security mechanism blocked execution

---

# 🔍 Important Observation

Normally:
- reflected script tags execute immediately

But here:
- the browser ignored the script completely

This strongly suggested:
- a Content Security Policy was active

---

## Screenshot 3 — Identifying the CSP Header

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(3).png?raw=true)

The request was captured:

```http
GET /?search=<script>alert(1)</script>
```

Inside the response headers:

```http
Content-Security-Policy:
default-src 'self';
object-src 'none';
script-src 'self';
style-src 'self';
report-uri /csp-report?token=
```

---

# 🔍 Full CSP Breakdown

---

# `default-src 'self';`

## Meaning

Fallback security rule.

It tells the browser:

```text
Only load resources from the same origin by default.
```

---

## Same Origin Means

Resources are allowed only from:
- same domain
- same protocol
- same port

Example:

```text
https://example.com
```

can only load resources from itself.

---

# `object-src 'none';`

## Meaning

Completely blocks old plugin-based content such as:
- Flash
- Java Applets
- Silverlight

---

## Why Important

These technologies historically caused many security vulnerabilities.

Setting:

```http
object-src 'none'
```

fully disables them.

---

# `script-src 'self';`

## Meaning

Only JavaScript files from the same website are allowed.

---

## Important Effect

This blocks:
- inline scripts
- injected `<script>alert(1)</script>`
- external malicious scripts

Because inline scripts are NOT trusted.

---

# 🔍 Why the Payload Failed

Your payload:

```html
<script>alert(1)</script>
```

was inline JavaScript.

But CSP allowed only:

```http
script-src 'self'
```

Meaning:
- browser only trusted scripts loaded as files from the same server
- inline scripts were blocked automatically

---

# `style-src 'self';`

## Meaning

Only CSS stylesheets from the same origin are allowed.

Blocks:
- inline styles
- external malicious CSS

---

# `report-uri /csp-report?token=`

## Meaning

If CSP violations occur:
- browser sends violation reports to this endpoint

Example:
- blocked script execution
- unauthorized resource loading

The browser automatically POSTs reports here.

---

# 🔍 Why report-uri Became Important

Notice:

```http
report-uri /csp-report?token=
```

The `token=` parameter looked injectable.

This became the CSP injection point.

---

## Screenshot 4 — Testing CSP Injection

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(4).png?raw=true)

A modified URL was tested:

```text
https://example.net/?search=<script>alert(1)</script>&token=1
```

Purpose:
- determine whether the token parameter modified the CSP header

---

## Screenshot 5 — Confirming CSP Injection

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(5).png?raw=true)

After capturing the response:

```http
Content-Security-Policy:
default-src 'self';
object-src 'none';
script-src 'self';
style-src 'self';
report-uri /csp-report?token=1
```

This confirmed:
- attacker input directly modified the CSP header
- CSP injection was possible

---

# 🔍 Why This is Dangerous

If attackers can inject new CSP directives:
- they may weaken or disable protections
- browsers will follow the modified policy

This can completely defeat CSP security.

---

## Screenshot 6 — Preparing CSP Bypass Payload

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(6).png?raw=true)

A new CSP directive was URL-encoded:

```text
; script-src-elem 'unsafe-inline'
```

Encoded version:

```text
%3b script-src-elem 'unsafe-inline'
```

---

# 🔍 Understanding `script-src-elem`

`script-src-elem` controls:
- `<script>` HTML elements specifically

Example:

```html
<script>alert(1)</script>
```

This directive applies directly to script tags.

---

# 🔍 Understanding `'unsafe-inline'`

Normally CSP blocks:

```html
<script>alert(1)</script>
```

because inline scripts are unsafe.

But:

```http
'unsafe-inline'
```

tells the browser:

```text
Allow inline JavaScript execution.
```

---

# ⚠ Why `'unsafe-inline'` is Dangerous

It effectively disables one of CSP’s strongest protections.

Attackers can then execute:
- inline scripts
- event handlers
- injected JavaScript payloads

---

# 🔍 What the Final Injection Did

The injected directive:

```http
script-src-elem 'unsafe-inline'
```

overrode the stricter CSP behavior.

This allowed:
- inline `<script>` execution
- successful XSS exploitation

---

## Screenshot 7 — Successful CSP Bypass

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(7).png?raw=true)

Final URL used:

```text
https://example.net/?search=<script>alert(1)</script>&token=%3b script-src-elem 'unsafe-inline'
```

After loading:
- browser accepted the modified CSP
- inline scripts became allowed
- JavaScript executed successfully

An alert popup appeared.

---

## Screenshot 8 — Lab Solved

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20Protected%20by%20CSP,%20with%20CSP%20Bypass/screenshots/lab13(8).png?raw=true)

After successful execution:
- the application displayed:

```text
Congratulations, you have solved the lab!
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- user input was inserted directly into the CSP header
- attacker-controlled CSP directives became possible
- browser trusted the modified CSP policy

This completely weakened CSP protection.

---

# 💥 Impact of CSP Bypass

An attacker can potentially:
- execute arbitrary JavaScript
- bypass XSS protections
- steal cookies
- hijack accounts
- manipulate webpage content
- fully exploit XSS vulnerabilities

CSP becomes ineffective once attacker-controlled directives are introduced.

---

# 🛡 Mitigation

To prevent CSP bypass:
- never include user input inside CSP headers
- sanitize all header values
- avoid dynamic CSP construction
- use strict CSP configurations
- avoid `'unsafe-inline'`
- use nonces or hashes instead

---

# 🔍 Recommended Secure CSP

Example secure policy:

```http
Content-Security-Policy:
default-src 'self';
script-src 'self' 'nonce-random123';
object-src 'none';
base-uri 'none';
```

This:
- blocks inline scripts
- allows only trusted scripts with matching nonces

---

# 🧠 Skills Learned

- Understanding Content Security Policy (CSP)
- CSP directives breakdown
- Browser CSP enforcement
- CSP injection vulnerabilities
- CSP bypass techniques
- Inline script restrictions
- Header manipulation
- Advanced XSS exploitation

---

# 🧰 Tools Used

- Burp Suite
- Burp Decoder
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how Content Security Policy protections can be bypassed when attacker-controlled input is reflected into CSP headers.

By identifying the injectable `token=` parameter, it was possible to:
- modify CSP directives
- weaken browser protections
- enable inline JavaScript execution
- bypass CSP restrictions successfully

Through this lab, I learned:
- how CSP works internally
- why inline scripts are blocked
- how browsers enforce CSP policies
- how dangerous dynamic CSP construction is
- how CSP bypass vulnerabilities occur

The lab was successfully exploited by injecting a malicious CSP directive that enabled inline script execution using:

```http
script-src-elem 'unsafe-inline'
```

thereby bypassing the original CSP restrictions completely.
