# DOM XSS in innerHTML Sink Using Source location.search

## 📌 Lab Overview

This lab demonstrates a **DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability using the dangerous JavaScript property:

```javascript
innerHTML
```

The vulnerability occurred because user-controlled input from the URL was inserted directly into the webpage DOM without proper sanitization.

---

# 🔍 What is DOM XSS?

DOM XSS occurs when:
- JavaScript running in the browser takes user-controlled input
- and inserts it into the webpage using unsafe DOM methods

Unlike reflected or stored XSS:
- the server may never directly process the malicious payload
- the vulnerability exists entirely in client-side JavaScript

---

# 🔍 Understanding Source and Sink

## Source

A **Source** is where attacker-controlled data enters the application.

In this lab:

```javascript
window.location.search
```

was the source because it extracted user input from the URL search parameter.

Example:

```text
?search=hello
```

---

## Sink

A **Sink** is a dangerous JavaScript function or property that inserts data into the webpage.

In this lab:

```javascript
innerHTML
```

was the sink because it directly inserted user-controlled input into the HTML page.

---

# 🔍 What is innerHTML?

`innerHTML` is a JavaScript property used to:
- read HTML content
- modify HTML content
- insert HTML dynamically into webpages

Example:

```javascript
element.innerHTML = "<h1>Hello</h1>";
```

The browser interprets the inserted content as real HTML.

If user-controlled input is inserted without sanitization:
- attackers can inject malicious HTML
- attackers can execute JavaScript

---

# 🎯 Objective

The goal of this lab was to exploit the DOM-based functionality and execute JavaScript inside the browser.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/DOM%20XSS%20in%20innerHTML%20Sink%20Using%20Source%20location.search/screenshots/lab5(1).png?raw=true)

The application initially displayed a normal webpage containing a search functionality.

The word:

```text
hello
```

was entered into the search field for testing.

---

## Screenshot 2 — Inspecting Reflection

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/DOM%20XSS%20in%20innerHTML%20Sink%20Using%20Source%20location.search/screenshots/lab5(2).png?raw=true)

While inspecting the page source, the reflected input became visible inside a span tag:

```html
<span id="searchMessage">hello</span>
```

This confirmed:
- user input was being inserted dynamically into the webpage
- the value was reflected inside HTML content

---

## Screenshot 3 — Inspecting Client-Side JavaScript

![Screenshot 3](screenshot-xss3.png)

A client-side JavaScript function responsible for the reflection was identified:

```javascript
function doSearchQuery(query) { 
    document.getElementById('searchMessage').innerHTML = query; 
} 

var query = (new URLSearchParams(window.location.search)).get('search'); 

if(query) { 
    doSearchQuery(query); 
}
```

---

# 🔍 Vulnerability Analysis

## Source

```javascript
window.location.search
```

This extracted user-controlled data from the URL.

---

## Sink

```javascript
innerHTML
```

This inserted the data directly into the webpage.

Because no sanitization occurred, attackers could inject arbitrary HTML elements.

---

# 🖼 Screenshot 4 — Inspecting searchMessage Element

![Screenshot 4](screenshot-xss4.png)

Searching for:

```text
searchMessage
```

inside the inspect page revealed the vulnerable HTML element again:

```html
<span id="searchMessage">hello</span>
```

This confirmed that:
- the JavaScript function was modifying this specific HTML element
- injected content would appear inside the span tag

---

## Screenshot 5 — Testing XSS Payloads

![Screenshot 5](screenshot-xss5.png)

The first payload tested was:

```html
<script>alert('xss')</script>
```

However, the payload did not execute.

Possible reason:
- browsers sometimes block dynamically injected `<script>` tags when inserted using `innerHTML`

---

# 🔍 Why `<script>` Did Not Execute

When using:

```javascript
innerHTML
```

many browsers:
- insert the `<script>` tag into the DOM
- but do not automatically execute it

This is a browser security behavior.

---

# 🔍 Bypassing the Restriction

A different payload was used:

```html
<img src=x onerror=alert('1')>
```

---

# 🔍 Payload Breakdown

```html
<img src=x
```

Creates an image element with an invalid image source.

---

```html
onerror=alert('1')
```

Triggers JavaScript execution when the image fails to load.

Because the image source was invalid:
- the `onerror` event triggered automatically
- JavaScript executed successfully

---

## Screenshot 6 — Successful DOM XSS Execution

![Screenshot 6](screenshot-xss6.png)

After loading the payload:
- browser attempted to load the invalid image
- image loading failed
- `onerror` event executed automatically
- an alert popup appeared

This confirmed successful DOM XSS exploitation.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- user-controlled input from the URL was trusted
- input was inserted into the DOM using `innerHTML`
- no sanitization or encoding was performed

Example vulnerable behavior:

```javascript
element.innerHTML = userInput;
```

Attackers can inject:
- HTML elements
- event handlers
- executable JavaScript

---

# 💥 Impact of DOM XSS

An attacker can potentially:
- steal session cookies
- hijack accounts
- manipulate webpage content
- execute malicious JavaScript
- redirect users to phishing websites
- perform unauthorized actions

---

# 🛡 Mitigation

To prevent DOM XSS:
- avoid using `innerHTML` with user input
- use safe DOM APIs like `textContent`
- sanitize user-controlled data
- implement Content Security Policy (CSP)
- validate and encode input properly

Safe alternative:

```javascript
element.textContent = userInput;
```

---

# 🧠 Skills Learned

- Understanding DOM-based XSS
- Identifying dangerous DOM sinks
- Understanding `innerHTML`
- Event handler exploitation
- Browser parsing behavior
- HTML injection techniques
- Client-side vulnerability analysis

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how unsafe use of `innerHTML` can lead to DOM-Based Cross-Site Scripting vulnerabilities.

By analyzing frontend JavaScript code, it was possible to identify:
- a vulnerable source (`window.location.search`)
- and a dangerous sink (`innerHTML`)

Although `<script>` tags did not execute automatically, the vulnerability was successfully exploited using an image error event handler payload.

Through this lab, I learned:
- how DOM XSS works in client-side JavaScript
- why `innerHTML` is dangerous
- how browsers handle dynamically injected scripts
- how event handlers can bypass certain browser restrictions

The lab was successfully exploited using an `onerror` event handler payload that triggered JavaScript execution automatically.
