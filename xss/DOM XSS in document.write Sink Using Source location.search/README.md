# DOM XSS in document.write Sink Using Source location.search

## 📌 Lab Overview

This lab demonstrates a **DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability.

Unlike normal Reflected or Stored XSS:
- the vulnerability exists entirely in client-side JavaScript
- the server does not directly process the malicious payload
- the browser itself becomes responsible for executing the attack

---

# 🔍 What is DOM XSS?

DOM XSS occurs when:
- JavaScript running inside the browser takes user-controlled input
- and inserts it into the webpage without proper sanitization

The vulnerability happens inside the **Document Object Model (DOM)** of the browser.

In DOM XSS:
- the malicious payload is processed on the client side
- unsafe JavaScript functions manipulate the webpage dynamically
- attackers exploit insecure JavaScript code

---

# 🔍 Understanding Source and Sink

## What is a Source?

A **Source** is a place where user-controlled data enters the application.

Examples:
- URL parameters
- search fields
- cookies
- document.referrer
- window.location

In this lab:

```javascript
window.location.search
```

was the source because it collected input from the URL search query.

---

## What is a Sink?

A **Sink** is a dangerous JavaScript function or DOM property that places data into the webpage.

Examples:
- `document.write()`
- `innerHTML`
- `eval()`
- `setTimeout()`

In this lab:

```javascript
document.write()
```

was the sink because it inserted user-controlled data directly into the HTML page.

---

# 🎯 Objective

The goal of this lab was to exploit the DOM-based JavaScript functionality and execute arbitrary JavaScript code inside the browser.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/DOM%20XSS%20in%20document.write%20Sink%20Using%20Source%20location.search/screenshots/lab4(1).png?raw=true)

The application initially displayed a normal webpage containing a search functionality.

At this stage:
- no payload was injected
- application behaved normally
- search feature was available for testing

---

## Screenshot 2 — Inspecting Client-Side JavaScript

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/DOM%20XSS%20in%20document.write%20Sink%20Using%20Source%20location.search/screenshots/lab4(2).png?raw=true)

After searching for:

```text
hello
```

the input reflected in the webpage heading.

While inspecting the page source, a client-side JavaScript function became visible:

```javascript
function trackSearch(query) { 
    document.write('<img src="/resources/images/tracker.gif?searchTerms=' + query + '">'); 
} 

var query = (new URLSearchParams(window.location.search)).get('search'); 

if(query) { 
    trackSearch(query); 
}
```

---

# 🔍 Vulnerability Analysis

## Source

```javascript
window.location.search
```

This collected user-controlled input directly from the URL parameter.

Example:

```text
?search=hello
```

---

## Sink

```javascript
document.write()
```

This inserted the input directly into the webpage HTML.

Because no sanitization or encoding was performed:
- attackers could inject malicious HTML or JavaScript
- browser would interpret the injected content as real code

---

# 🖼 Screenshot 3 — Exploiting the DOM XSS

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/DOM%20XSS%20in%20document.write%20Sink%20Using%20Source%20location.search/screenshots/lab4(3).png?raw=true)

The following payload was used:

```html
"><script>alert('xss')</script>
```

---

# 🔍 Payload Breakdown

```html
">
```

This sequence:
- closes the current HTML attribute
- escapes from the existing `src=""` attribute

---

```html
<script>alert('xss')</script>
```

This starts a new script tag and executes JavaScript inside the browser.

Because the payload was inserted directly into the DOM using `document.write()`, the browser executed the injected script successfully.

An alert popup appeared, confirming successful DOM XSS exploitation.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- user-controlled input was taken from the URL
- input was inserted into HTML using `document.write()`
- no sanitization or encoding was performed

Example vulnerable behavior:

```javascript
document.write('<img src="' + userInput + '">');
```

This allowed attackers to break out of HTML attributes and inject arbitrary JavaScript.

---

# 💥 Impact of DOM XSS

An attacker can potentially:
- steal session cookies
- hijack accounts
- execute malicious JavaScript
- manipulate webpage content
- redirect users to phishing websites
- perform actions on behalf of victims

DOM XSS is especially dangerous because:
- it happens entirely in the browser
- many scanners fail to detect it
- vulnerabilities often hide inside frontend JavaScript

---

# 🛡 Mitigation

To prevent DOM XSS:
- avoid dangerous sinks like `document.write()`
- sanitize user-controlled input
- use safe DOM APIs
- encode data before inserting into HTML
- implement Content Security Policy (CSP)
- avoid directly concatenating user input into HTML

---

# 🧠 Skills Learned

- Understanding DOM-based XSS
- Identifying JavaScript sources and sinks
- Client-side vulnerability analysis
- HTML attribute breakout techniques
- Browser-side payload execution
- DOM inspection and JavaScript auditing

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how insecure client-side JavaScript can lead to DOM-Based Cross-Site Scripting vulnerabilities.

By analyzing frontend JavaScript code, it was possible to identify:
- a vulnerable source (`window.location.search`)
- and a dangerous sink (`document.write()`)

Using an attribute breakout payload, arbitrary JavaScript execution was achieved directly inside the browser DOM.

Through this lab, I learned:
- how DOM XSS differs from reflected and stored XSS
- how sources and sinks work
- how unsafe DOM manipulation leads to vulnerabilities
- how attackers exploit client-side JavaScript

The lab was successfully exploited by breaking out of an HTML attribute and injecting a malicious script payload.
