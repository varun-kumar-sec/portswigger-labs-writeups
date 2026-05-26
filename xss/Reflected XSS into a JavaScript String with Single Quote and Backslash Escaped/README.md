# Reflected XSS into a JavaScript String with Single Quote and Backslash Escaped

## 📌 Lab Overview

This lab demonstrates a **Reflected Cross-Site Scripting (XSS)** vulnerability inside a JavaScript context.

Unlike normal HTML-based XSS:
- user input was reflected directly inside JavaScript code
- the payload executed inside a `<script>` block
- exploitation required breaking out of an existing JavaScript string

This type of vulnerability is dangerous because applications often trust JavaScript contexts more than HTML contexts.

---

# 🔍 What is JavaScript Context XSS?

JavaScript Context XSS occurs when:
- user-controlled input is inserted inside JavaScript code
- without proper sanitization or escaping

Example vulnerable behavior:

```javascript
var search = 'USER_INPUT';
```

If attackers can break out of the string:
- they can inject arbitrary JavaScript
- browser executes the malicious code immediately

---

# 🎯 Objective

The goal of this lab was to inject and execute JavaScript by escaping an existing JavaScript string inside a `<script>` tag.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20a%20JavaScript%20String%20with%20Single%20Quote%20and%20Backslash%20Escaped/screenshots/lab6(1).png?raw=true)

The application initially displayed a normal webpage containing a search functionality.

The word:

```text
hello
```

was entered into the search field for testing.

---

## Screenshot 2 — Inspecting Client-Side JavaScript

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20a%20JavaScript%20String%20with%20Single%20Quote%20and%20Backslash%20Escaped/screenshots/lab6(2).png?raw=true)

After searching for:

```text
hello
```

the page source was inspected.

During inspection, the following client-side JavaScript code was identified:

```javascript
var searchTerms = 'hello'; 
document.write('<img src="/resources/images/tracker.gif?searchTerms=' + encodeURIComponent(searchTerms) + '">');
```

---

# 🔍 Vulnerability Analysis

The user-controlled input:

```text
hello
```

was directly inserted into a JavaScript string:

```javascript
var searchTerms = 'hello';
```

This meant:
- the search parameter was reflected inside JavaScript code
- breaking out of the string could allow arbitrary JavaScript execution

---

# 🖼 Screenshot 3 — Exploiting the Vulnerability

![Screenshot 3](screenshot-xss3.png)

An interesting observation was made:
- the vulnerable JavaScript already existed inside `<script>` tags

Because of this, the following payload was used:

```html
</script><script>alert(1)</script>
```

---

# 🔍 Payload Breakdown

## Closing the Existing Script

```html
</script>
```

This closes the original JavaScript block.

---

## Starting a New Script

```html
<script>alert(1)</script>
```

This creates a brand-new JavaScript block that executes independently.

---

# 🔍 Why This Worked

The browser interpreted the payload as:

```html
<script>
var searchTerms = '
</script>

<script>
alert(1)
</script>
```

So:
- the original script was terminated
- a new malicious script block was created
- browser executed the injected JavaScript successfully

---

## Screenshot 4 — Successful XSS Execution

![Screenshot 4](screenshot-xss4.png)

After loading the payload:
- an alert popup appeared
- JavaScript executed successfully
- the XSS vulnerability was confirmed

While inspecting the page source, the following structure became visible:

```html
<h1>
0 search results for '</script><script>alert(1)</script>'
</h1>

<script>
var searchTerms = '</script>
<script>alert(1)</script>
```

This clearly demonstrated:
- how the original script block was broken
- how the malicious script was injected
- how the browser interpreted the payload as executable JavaScript

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- user-controlled input was inserted directly into JavaScript
- no proper context-aware escaping was performed
- the application trusted untrusted input inside `<script>` blocks

Example vulnerable behavior:

```javascript
var searchTerms = 'USER_INPUT';
```

Attackers can break out of the string and inject arbitrary JavaScript.

---

# 💥 Impact of JavaScript Context XSS

An attacker can potentially:
- steal session cookies
- hijack accounts
- perform phishing attacks
- execute arbitrary JavaScript
- manipulate webpage content
- perform actions on behalf of victims

JavaScript-context XSS is especially dangerous because:
- code executes directly inside existing scripts
- payloads can bypass some HTML-based protections

---

# 🛡 Mitigation

To prevent JavaScript-context XSS:
- perform context-aware output encoding
- escape special JavaScript characters properly
- avoid inserting raw user input inside scripts
- use secure templating frameworks
- implement Content Security Policy (CSP)

Safe example:

```javascript
var searchTerms = JSON.stringify(userInput);
```

---

# 🧠 Skills Learned

- Understanding JavaScript-context XSS
- Breaking out of JavaScript strings
- Script tag injection techniques
- Client-side JavaScript analysis
- Browser parsing behavior
- Context-aware payload crafting

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improperly handled user input inside JavaScript code can lead to Reflected Cross-Site Scripting vulnerabilities.

By identifying that the input was reflected inside a `<script>` block, it was possible to break out of the existing script and inject a new malicious JavaScript payload.

Through this lab, I learned:
- how JavaScript-context XSS works
- how browsers parse script tags
- how attackers escape existing JavaScript blocks
- why context-aware escaping is important
- how client-side JavaScript can become dangerous when user input is trusted

The lab was successfully exploited by closing the original script block and injecting a new malicious `<script>` tag that executed JavaScript successfully.
