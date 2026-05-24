# Reflected XSS into Attribute with Angle Brackets HTML-Encoded

## 📌 Lab Overview

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability inside an HTML attribute context.

Unlike normal reflected XSS:
- the application encoded angle brackets (`<` and `>`)
- direct `<script>` payloads were blocked
- JavaScript execution required a different injection technique

The vulnerability existed because user input was reflected inside an HTML input attribute without proper sanitization.

---

# 🎯 Objective

The goal of this lab was to bypass HTML encoding protections and execute JavaScript inside the browser.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20Attribute%20with%20Angle%20Brackets%20HTML-Encoded/screenshots/lab2(1).png?raw=true)

The application initially displayed a normal webpage containing a search functionality.

At this stage:
- no payload was injected
- application behaved normally
- search feature was available for testing

---

## Screenshot 2 — Testing Reflection

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20Attribute%20with%20Angle%20Brackets%20HTML-Encoded/screenshots/lab2(2).png?raw=true)

The word:

```text
hello
```

was entered into the search field.

After searching, the input reflected inside the webpage heading:

```html
0 search result for 'hello'
```

This confirmed that:
- user input was reflected back into the response
- reflection existed in the application

---

## Screenshot 3 — Inspecting the Reflection

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20Attribute%20with%20Angle%20Brackets%20HTML-Encoded/screenshots/lab2(3).png?raw=true)

Using browser inspect tools, the reflected input was searched inside the HTML source.

The first reflection appeared inside the heading:

```html
<h1>0 search result for 'hello'</h1>
```

This helped identify where user input was being inserted into the page.

---

## Screenshot 4 — Testing Basic XSS Payload

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20Attribute%20with%20Angle%20Brackets%20HTML-Encoded/screenshots/lab2(4).png?raw=true)

The following payload was tested:

```html
<script>alert('xss')</script>
```

After inspecting the page source, it became visible that angle brackets were HTML-encoded:

```html
&lt;script&gt;alert('xss')&lt;/script&gt;
```

Inside the input field:

```html
<input type="text" placeholder="Search the blog..." name="search" value="&lt;script&gt;alert('xss')&lt;/script&gt;">
```

This prevented the browser from interpreting the payload as an actual `<script>` tag.

---

# 🔍 Understanding HTML Encoding (`&lt;` and `&gt;`)

## Simple Meaning

The application converted:

```html
<
>
```

into:

```html
&lt;
&gt;
```

This process is called **HTML Encoding**.

---

## Why Applications Do This

Applications encode angle brackets to prevent browsers from interpreting user input as real HTML or JavaScript.

Example:

### Original Payload

```html
<script>alert(1)</script>
```

### Encoded Version

```html
&lt;script&gt;alert(1)&lt;/script&gt;
```

Now the browser treats it as normal text instead of executable code.

---

# 🖼 Screenshot 5 — Bypassing the Filter Using Event Handlers

![Screenshot 5](screenshot-xss5.png)

Since angle brackets were blocked, a new payload was crafted without using `<script>` tags:

```html
a" onfocus="alert('xss')" autofocus="alert('xss')
```

### Payload Breakdown

```html
a"
```

- closes the current `value=""` attribute

---

```html
onfocus="alert('xss')"
```

- injects a new event handler

---

```html
autofocus
```

- automatically focuses the input field when the page loads

This caused the browser to automatically trigger the `onfocus` event.

---

# 🔍 Understanding Event Handlers

## Simple Meaning

Event handlers are special HTML attributes that execute JavaScript when specific actions happen.

Examples:

| Event Handler | Trigger |
|---|---|
| `onclick` | User clicks |
| `onmouseover` | Mouse hovers |
| `onfocus` | Input field gets focus |
| `onload` | Page loads |

---

## Example

```html
<input onfocus="alert(1)">
```

When the input field receives focus:
- JavaScript executes automatically

---

## Why Event Handlers Worked Here

The application blocked:
- `<script>` tags
- angle brackets

But it still allowed:
- HTML attributes
- event handlers

Because event handlers do not always require angle brackets for execution inside attribute injection contexts, the payload successfully bypassed the filter.

---

## Screenshot 6 — Successful XSS Execution

![Screenshot 6](screenshot-xss6.png)

After loading the page:
- the input field automatically received focus
- the `onfocus` event executed
- JavaScript alert popup appeared

This confirmed:
- successful attribute injection
- successful JavaScript execution
- reflected XSS vulnerability

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The application reflected user input inside an HTML attribute without proper sanitization.

Although angle brackets were encoded, dangerous attributes such as:
- `onfocus`
- `onclick`
- `onmouseover`

were still allowed.

This allowed attackers to inject executable JavaScript through HTML attributes.

---

# 💥 Impact of Reflected XSS

An attacker can potentially:
- steal session cookies
- hijack user accounts
- redirect victims to malicious websites
- perform phishing attacks
- execute unauthorized actions
- manipulate webpage content

---

# 🛡 Mitigation

To prevent attribute-based XSS:
- perform context-aware output encoding
- sanitize dangerous HTML attributes
- block inline JavaScript execution
- implement Content Security Policy (CSP)
- validate and escape user-controlled input properly

---

# 🧠 Skills Learned

- Understanding attribute-based XSS
- HTML encoding behavior
- Event handler exploitation
- Attribute injection techniques
- Context-aware payload crafting
- Browser parsing behavior
- XSS filter bypass methods

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how Reflected XSS can still occur even when angle brackets are HTML-encoded.

Although direct `<script>` injection was blocked, it was possible to bypass the filter by injecting malicious event handlers inside an HTML attribute context.

Through this lab, I learned:
- how HTML encoding works
- why encoding alone is not always sufficient
- how attribute injection vulnerabilities occur
- how event handlers can execute JavaScript
- how attackers bypass partial XSS protections

The lab was successfully exploited using an `onfocus` event handler payload that triggered JavaScript execution automatically.
