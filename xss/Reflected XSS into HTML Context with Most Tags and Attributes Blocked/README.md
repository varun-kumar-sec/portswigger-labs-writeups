# Reflected XSS into HTML Context with Most Tags and Attributes Blocked

## 📌 Lab Overview

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability where:
- most HTML tags were blocked
- most event handlers were filtered
- direct `<script>` injection was prevented

The challenge of this lab was to:
- identify allowed HTML tags
- identify allowed event handlers
- craft a working payload using the remaining allowed functionality

This lab focused heavily on:
- XSS filter bypass techniques
- HTML tag enumeration
- event handler discovery
- payload customization

---

# 🎯 Objective

The goal of this lab was to bypass the application's filtering protections and execute JavaScript successfully inside the browser.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(1).png?raw=true)

The application initially displayed a normal webpage containing a search functionality.

The word:

```text
hello
```

was entered into the search field for testing.

---

## Screenshot 2 — Reflection Testing

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(2).png?raw=true)

After searching:
- the word `hello` reflected in the webpage heading

Example:

```html
0 search result for 'hello'
```

While inspecting the page:
- nothing immediately vulnerable was identified
- no dangerous JavaScript reflection was visible

This indicated that:
- further testing would be required
- the application likely had filtering protections enabled

---

## Screenshot 3 — Testing Basic XSS Payload

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(3).png?raw=true)

The following payload was tested:

```html
<script>alert(1)</script>
```

This was used to test whether standard script injection was possible.

---

## Screenshot 4 — Filter Response

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(4).png?raw=true)

After submitting the payload:
- the application blocked the request
- a message appeared:

```text
Tag is not allowed
```

This confirmed:
- the application was filtering HTML tags
- direct `<script>` execution was not possible

---

## Screenshot 5 — Capturing the Request

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(5).png?raw=true)

The blocked request was captured inside Burp Suite:

```http
GET /?search=%3Cscript%3E
```

This request became the base for further testing.

---

## Screenshot 6 — Using PortSwigger XSS Cheat Sheet

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(6).png?raw=true)

The PortSwigger XSS Cheat Sheet was used to:
- copy all possible HTML tags
- test which tags were allowed by the filter

This is a common real-world methodology for bypassing restrictive XSS filters.

---

## Screenshot 7 — Tag Enumeration Using Intruder

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(7).png?raw=true)

The request was sent to Burp Intruder.

Payload position:

```http
GET /?search=%3C$script$%3E
```

All possible HTML tags from the cheat sheet were pasted into Intruder payloads.

Purpose:
- identify which tags bypass the filter
- discover usable HTML injection points

---

## Screenshot 8 — Discovering Allowed Tags

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20HTML%20Context%20with%20Most%20Tags%20and%20Attributes%20Blocked/screenshots/lab7(8).png?raw=true)

Intruder results showed that only two tags were accepted:

```text
<body>
<xss>
```

The important discovery was:

```html
<body>
```

because it supports multiple JavaScript event handlers.

---

# 🔍 Why Allowed Tags Matter

Even if `<script>` is blocked:
- event handlers can still execute JavaScript
- attackers only need one executable context

Example:

```html
<body onresize="alert(1)">
```

---

## Screenshot 9 — Event Handler Enumeration

![Screenshot 9](screenshot-xss9.png)

The request was modified again:

```http
GET /?search=<body $a$=1 >
```

The payload placeholder:

```text
$a$
```

was used to test all possible event handlers from the XSS cheat sheet.

Purpose:
- identify which events were allowed with the `<body>` tag

---

## Screenshot 10 — Discovering Valid Events

![Screenshot 10](screenshot-xss10.png)

Intruder results revealed multiple working event handlers.

The most interesting valid handler was:

```html
onresize
```

This became the primary exploitation method.

---

# 🔍 What is onresize?

`onresize` is an HTML event handler that executes JavaScript whenever the browser window changes size.

Example:

```html
<body onresize="alert(1)">
```

When the window resizes:
- JavaScript executes automatically

---

## Screenshot 11 — Crafting Final Payload

![Screenshot 11](screenshot-xss11.png)

Using the PortSwigger XSS Cheat Sheet, the following payload was selected:

```html
<body onresize="print()">
```

---

# 🔍 Payload Breakdown

## `<body>`

Allowed HTML tag.

---

## `onresize`

Event handler triggered during window resizing.

---

## `print()`

JavaScript function that opens the browser print dialog.

This confirmed successful JavaScript execution.

---

## Screenshot 12 — Executing the Payload

![Screenshot 12](screenshot-xss12.png)

The payload was inserted into the search field.

When the browser window resized:
- the `onresize` event triggered
- JavaScript executed successfully
- the print dialog appeared

This confirmed successful XSS execution.

---

## Screenshot 13 — Triggering the Event

![Screenshot 13](screenshot-xss13.png)

The browser was resized using:
- Developer Tools (`F12`)
- manual browser resizing

This triggered:
- the `onresize` event
- successful JavaScript execution

---

## Screenshot 14 — Preparing the Exploit URL

![Screenshot 14](screenshot-xss14.png)

The full vulnerable URL was copied:

```text
https://0a3e00f5049b50d3801e71e3003c00dc.web-security-academy.net/?search=<body+onresize%3D"print()">
```

This URL contained the malicious XSS payload.

---

## Screenshot 15 — Exploit Server Delivery

![Screenshot 15](screenshot-xss15.png)

The provided exploit server was used to automate victim interaction.

The following payload was inserted into the exploit server body:

```html
<iframe src="https://0a3e00f5049b50d3801e71e3003c00dc.web-security-academy.net/?search=%3Cbody+onresize%3D%22print%28%29%22%3E" onload=this.style.width='100px'>
```

---

# 🔍 Why the iframe Worked

The iframe:
- automatically loaded the vulnerable page
- resized itself using:

```javascript
onload=this.style.width='100px'
```

This triggered:
- the `onresize` event inside the victim page
- automatic JavaScript execution

The exploit was then delivered to the victim.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- dangerous HTML tags were partially filtered
- dangerous event handlers were still allowed
- user input was reflected into HTML without safe sanitization

Even strict filters become dangerous if:
- a single executable tag survives
- a single executable event handler remains allowed

---

# 💥 Impact of Reflected XSS

An attacker can potentially:
- execute arbitrary JavaScript
- steal session cookies
- hijack user accounts
- perform phishing attacks
- redirect victims to malicious websites
- perform actions on behalf of victims

This lab also demonstrated:
- real-world XSS filter bypass techniques

---

# 🛡 Mitigation

To prevent advanced XSS bypasses:
- implement strict allowlists
- sanitize dangerous attributes
- remove inline JavaScript execution
- implement Content Security Policy (CSP)
- avoid reflecting raw user input into HTML

---

# 🧠 Skills Learned

- Advanced XSS filter bypass
- Tag enumeration using Burp Intruder
- Event handler discovery
- HTML context exploitation
- Payload customization
- Browser event exploitation
- Automated victim delivery
- Using exploit servers
- Real-world XSS methodology

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Firefox Developer Tools
- PortSwigger XSS Cheat Sheet
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how Cross-Site Scripting vulnerabilities can still exist even when most HTML tags and attributes are blocked.

By systematically enumerating:
- allowed HTML tags
- allowed event handlers

it was possible to bypass the application's filtering protections and achieve JavaScript execution.

Through this lab, I learned:
- how attackers bypass restrictive XSS filters
- how event handlers become alternative execution vectors
- how Burp Intruder can automate payload testing
- how browser events can trigger JavaScript execution
- how exploit servers are used to deliver payloads automatically

The lab was successfully exploited using the `<body>` tag combined with the `onresize` event handler and automated iframe resizing.
