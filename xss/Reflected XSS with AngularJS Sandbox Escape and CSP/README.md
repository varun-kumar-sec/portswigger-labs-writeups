# Reflected XSS with AngularJS Sandbox Escape and CSP

## 📌 Lab Overview

This lab demonstrates how AngularJS expressions can be abused to bypass both:
- AngularJS sandbox protections
- Content Security Policy (CSP)

The vulnerability occurred because:
- AngularJS expressions were evaluated dynamically
- CSP still allowed AngularJS directives
- attacker-controlled AngularJS expressions executed inside the page
- AngularJS sandbox protections could be escaped

This lab focused on:
- AngularJS sandbox escape
- CSP bypass
- AngularJS directives
- DOM-based JavaScript execution
- event-based XSS exploitation

---

# 🔍 What is AngularJS Sandbox?

AngularJS introduced a security mechanism called:

```text
Sandbox
```

The purpose of the sandbox was:
- restrict dangerous JavaScript execution
- prevent access to sensitive browser objects
- block arbitrary code execution

AngularJS tried to stop access to:
- `window`
- `document`
- JavaScript constructors
- dangerous functions like `alert()`

---

# ⚠ Why the Sandbox Failed

Older AngularJS versions contained bypass techniques.

Attackers discovered methods to:
- manipulate AngularJS expressions
- abuse filters
- access browser functions indirectly
- escape AngularJS restrictions

This technique is called:

```text
AngularJS Sandbox Escape
```

---

# 🔍 What is CSP?

CSP (Content Security Policy) is a browser security mechanism that:
- restricts JavaScript execution
- blocks inline scripts
- prevents malicious resources from loading

Example:

```http
Content-Security-Policy: script-src 'self'
```

Meaning:
- only JavaScript from the same website can execute

Purpose:
- reduce XSS attacks

---

# ⚠ Why CSP Was Bypassed in This Lab

Although inline JavaScript was blocked:
- AngularJS directives were still allowed
- AngularJS internally evaluated expressions
- attackers abused AngularJS behavior instead of traditional `<script>` tags

---

# 🎯 Objective

The goal of this lab was to:
- bypass AngularJS sandbox protections
- bypass CSP restrictions
- execute arbitrary JavaScript
- deliver the exploit to the victim

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20and%20CSP/screenshots/lab15(1).png?raw=true)

The application initially displayed:
- a normal webpage
- a search functionality

---

## Screenshot 2 — Finding the Payload

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20and%20CSP/screenshots/lab15(2).png?raw=true)

Since the lab required:
- AngularJS sandbox escape
- CSP bypass

A payload from the PortSwigger XSS Cheat Sheet was used:

```html
<input id=x ng-focus=$event.composedPath()|orderBy:'(z=alert)(document.cookie)'#x>
```

---

# 🔍 Full Payload Breakdown

---

# `<input id=x ... >`

Creates an HTML input element.

---

# `id=x`

Assigns the element an ID:

```html
id="x"
```

This is important later for automatic focus.

---

# `ng-focus=`

`ng-focus` is an AngularJS directive.

It behaves similarly to JavaScript:

```javascript
onfocus
```

Meaning:
- when the input gains focus
- AngularJS executes the expression

---

# 🔍 What is Focus?

Focus means:
- clicking inside an input field
- selecting an element
- cursor becoming active

Example:

```html
<input autofocus>
```

The browser automatically focuses it.

---

# `$event.composedPath()`

This accesses the browser event object.

Purpose:
- retrieve event path information
- indirectly access browser internals

This helps bypass AngularJS restrictions.

---

# `|orderBy:`

AngularJS supports filters like:

```javascript
orderBy
```

Normally:
- used for sorting arrays

But attackers abuse it because:
- AngularJS evaluates expressions inside filters

---

# `'(z=alert)(document.cookie)'`

This is the actual JavaScript execution.

---

# 🔍 Breakdown

## `z=alert`

Stores:

```javascript
alert
```

inside variable:

```javascript
z
```

---

## `(document.cookie)`

Executes:

```javascript
alert(document.cookie)
```

Result:
- browser cookies are displayed

---

# ⚠ Why This Bypasses CSP

CSP blocked:
- inline `<script>` tags

But AngularJS itself:
- evaluated expressions internally

So no direct `<script>` execution was needed.

---

# `#x`

This automatically focuses the input element.

Equivalent behavior:

```javascript
location.hash = "#x"
```

Purpose:
- automatically trigger `ng-focus`
- execute payload without user interaction

---

## Screenshot 3 — Injecting the Payload

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20and%20CSP/screenshots/lab15(3).png?raw=true)

The payload was inserted into the search bar.

The URL now contained:
- AngularJS directive injection
- sandbox escape logic
- CSP bypass payload

---

## Screenshot 4 — Successful XSS Execution

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20and%20CSP/screenshots/lab15(4).png?raw=true)

After loading the payload:
- AngularJS evaluated the expression
- the sandbox was bypassed
- CSP protections were bypassed
- browser cookies were displayed

A popup containing:

```javascript
document.cookie
```

was successfully executed.

---

## Screenshot 5 — Copying the Exploit URL

![Screenshot 5](screenshot-xss5.png)

After successful execution:
- the malicious URL was copied

Purpose:
- deliver the exploit to the victim

---

## Screenshot 6 — Exploit Server Delivery

![Screenshot 6](screenshot-xss6.png)

Inside the PortSwigger exploit server, the following script was created:

```html
<script>
location="https://0abe00440329e3c38141c0c700ee009c.web-security-academy.net/?search=%3Cinput+id%3Dx+ng-focus%3D%24event.composedPath()|orderBy%3A%27(z%3Dalert)(document.cookie)%27%3E#x";
</script>
```

Purpose:
- automatically redirect the victim
- execute the AngularJS payload
- trigger XSS automatically

After delivering the exploit to the victim:
- the lab was successfully solved

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- AngularJS expressions were evaluated dynamically
- user-controlled input reached AngularJS directives
- CSP trusted AngularJS execution
- AngularJS sandbox protections were insufficient

---

# 💥 Impact of This Vulnerability

An attacker can potentially:
- steal cookies
- hijack sessions
- perform account takeover
- bypass CSP protections
- execute arbitrary JavaScript
- compromise users

---

# 🛡 Mitigation

To prevent AngularJS sandbox escape vulnerabilities:
- never evaluate user-controlled AngularJS expressions
- disable unsafe AngularJS directives
- upgrade AngularJS versions
- implement strict CSP
- sanitize user-controlled input
- avoid dangerous client-side expression parsing

---

# 🧠 Skills Learned

- AngularJS sandbox concepts
- CSP bypass techniques
- AngularJS directives
- Event-based XSS
- DOM-based JavaScript execution
- AngularJS expression injection
- Automatic focus triggering
- Exploit delivery

---

# 🧰 Tools Used

- Burp Suite
- Browser Developer Tools
- PortSwigger XSS Cheat Sheet
- PortSwigger Exploit Server

---

# ✅ Conclusion

This lab demonstrated how AngularJS expressions can be abused to bypass both:
- AngularJS sandbox protections
- Content Security Policy (CSP)

By injecting a malicious AngularJS directive:
- arbitrary JavaScript execution became possible
- browser cookies were accessed
- CSP restrictions were bypassed successfully

Through this lab, I learned:
- how AngularJS evaluates expressions
- how sandbox escape vulnerabilities occur
- how CSP can still be bypassed
- how AngularJS directives become dangerous when combined with user input

The lab was successfully solved by abusing AngularJS expression evaluation and bypassing both sandbox and CSP protections.
