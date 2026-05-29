# Reflected XSS into a Template Literal with Angle Brackets, Single, Double Quotes, Backslash and Backticks Unicode-Escaped

## 📌 Lab Overview

This lab demonstrates how JavaScript template literals can become vulnerable to **Reflected Cross-Site Scripting (XSS)**.

The vulnerability occurred because:
- user-controlled input was inserted directly into a JavaScript template literal
- the application failed to sanitize JavaScript expression syntax
- JavaScript expressions inside the template literal were evaluated dynamically

This lab focused on:
- JavaScript template literals
- JavaScript expression injection
- reflected XSS
- client-side JavaScript execution

---

# 🔍 What is a Template Literal?

A template literal is a JavaScript string written using:

```javascript
`
```

(backticks)

instead of:
- single quotes `'`
- double quotes `"`

---

# ✅ Normal JavaScript String

```javascript
var name = "hello";
```

---

# ✅ Template Literal

```javascript
var name = `hello`;
```

---

# 🔍 Why Template Literals are Special

Template literals allow:
- multiline strings
- embedded JavaScript expressions

using:

```javascript
${...}
```

Example:

```javascript
var user = "carlos";

var msg = `Hello ${user}`;
```

Result:

```text
Hello carlos
```

---

# ⚠ Why This Can Become Dangerous

If attacker-controlled input reaches:

```javascript
${...}
```

then JavaScript code inside it executes automatically.

Example:

```javascript
${alert(1)}
```

This immediately executes:

```javascript
alert(1)
```

---

# 🎯 Objective

The goal of this lab was to:
- identify the vulnerable JavaScript template literal
- inject a malicious JavaScript expression
- execute arbitrary JavaScript successfully

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20a%20Template%20Literal%20with%20Angle%20Brackets,%20Single,%20Double%20Quotes,%20Backslash%20and%20Backticks%20Unicode-Escaped/screenshots/lab15(1).png?raw=true)

The application initially displayed:
- a normal webpage
- a search functionality

The word:

```text
hello
```

was entered into the search box.

---

## Screenshot 2 — Discovering the Vulnerable Template Literal

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20a%20Template%20Literal%20with%20Angle%20Brackets,%20Single,%20Double%20Quotes,%20Backslash%20and%20Backticks%20Unicode-Escaped/screenshots/lab15(2).png?raw=true)

While inspecting the page and searching for the word:

```text
hello
```

the following JavaScript code was discovered:

```javascript
var message = `0 search results for 'hello'`;

document.getElementById('searchMessage').innerText = message;
```

---

# 🔍 Code Breakdown

---

# `var message =`

Creates a JavaScript variable named:

```javascript
message
```

---

# Backticks `` ` ``

The string was written using:

```javascript
`
```

which means:
- this is a JavaScript template literal

---

# `0 search results for 'hello'`

The user-controlled input:

```text
hello
```

was being inserted directly into the template literal.

---

# `document.getElementById('searchMessage')`

Selects the HTML element with ID:

```html
searchMessage
```

---

# `.innerText = message`

Displays the message on the webpage.

---

# ⚠ Important Observation

The application escaped:
- angle brackets
- quotes
- backslashes
- backticks

But it failed to sanitize:

```javascript
${...}
```

which still works inside template literals.

---

# ⚠ Vulnerability Root Cause

The vulnerability occurred because:
- user input was inserted directly into a template literal
- JavaScript expressions inside `${}` were still evaluated

---

## Screenshot 3 — Injecting the Payload

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20a%20Template%20Literal%20with%20Angle%20Brackets,%20Single,%20Double%20Quotes,%20Backslash%20and%20Backticks%20Unicode-Escaped/screenshots/lab15(3).png?raw=true)

The following payload was inserted into the search field:

```javascript
${alert(1)}
```

---

# 🔍 Why This Payload Works

Inside template literals:

```javascript
${...}
```

means:

```text
"Execute the JavaScript code inside this block."
```

So when the application created:

```javascript
var message = `0 search results for '${alert(1)}'`;
```

JavaScript interpreted it as:

```javascript
alert(1)
```

and immediately executed the code.

---

# 🔍 Simple Example

## Normal Template Literal

```javascript
var name = "wiener";

`Hello ${name}`
```

Result:

```text
Hello wiener
```

---

## Malicious Template Literal

```javascript
`${alert(1)}`
```

Result:
- JavaScript executes `alert(1)`
- popup appears

---

# ⚠ Why Escaping Failed

The application escaped:
- `<script>`
- quotes
- backticks

But template literals evaluate:

```javascript
${...}
```

as JavaScript expressions directly.

So attackers do not need:
- `<script>`
- quotes
- HTML injection

Only:

```javascript
${...}
```

is enough.

---

## Screenshot 4 — Successful XSS Execution

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20into%20a%20Template%20Literal%20with%20Angle%20Brackets,%20Single,%20Double%20Quotes,%20Backslash%20and%20Backticks%20Unicode-Escaped/screenshots/lab15(4).png?raw=true)

After submitting the payload:
- JavaScript expression execution occurred
- a popup displaying:

```text
1
```

appeared successfully

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- user-controlled input was inserted into JavaScript template literals
- expressions inside `${}` were evaluated dynamically
- sanitization did not block template literal expressions

---

# 💥 Impact of This Vulnerability

An attacker can potentially:
- execute arbitrary JavaScript
- steal cookies
- hijack sessions
- manipulate webpage content
- perform phishing attacks
- compromise users

---

# 🛡 Mitigation

To prevent template literal XSS vulnerabilities:
- never insert unsanitized user input into template literals
- properly escape `${}`
- avoid dynamic JavaScript generation
- use safe templating frameworks
- sanitize all user-controlled input
- implement strict Content Security Policy (CSP)

---

# 🧠 Skills Learned

- Understanding JavaScript template literals
- Expression injection
- Reflected XSS
- Client-side JavaScript execution
- JavaScript context analysis
- Template literal exploitation

---

# 🧰 Tools Used

- Browser Developer Tools
- Burp Suite
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how JavaScript template literals can become vulnerable to reflected XSS when attacker-controlled input is inserted directly into the template literal.

By abusing:

```javascript
${...}
```

it was possible to:
- inject JavaScript expressions
- execute arbitrary JavaScript
- trigger XSS successfully

Through this lab, I learned:
- how template literals work internally
- how JavaScript expressions inside `${}` are evaluated
- why template literals become dangerous when combined with unsanitized user input

The lab was successfully solved by injecting a malicious JavaScript expression into a vulnerable template literal.
