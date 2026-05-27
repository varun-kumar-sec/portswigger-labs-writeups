# Reflected DOM XSS

## 📌 Lab Overview

This lab demonstrates a **Reflected DOM-Based Cross-Site Scripting (DOM XSS)** vulnerability caused by the dangerous JavaScript function:

```javascript
eval()
```

The vulnerability occurred because:
- user-controlled input from the search parameter was reflected into a JSON response
- the frontend JavaScript processed that response using `eval()`
- attackers could break the JSON structure and inject arbitrary JavaScript

This lab focused heavily on:
- JavaScript parsing
- JSON injection
- escaping characters
- dangerous JavaScript execution functions

---

# 🔍 What is Reflected DOM XSS?

Reflected DOM XSS occurs when:
- attacker-controlled input is reflected dynamically
- frontend JavaScript processes that input insecurely
- JavaScript execution happens inside the browser DOM

Unlike Stored DOM XSS:
- the payload is not permanently stored
- the attack occurs immediately through a crafted URL or request

---

# 🔍 What is eval()?

`eval()` is a dangerous JavaScript function that:
- executes strings as JavaScript code

Example:

```javascript
eval("alert(1)")
```

This directly executes:

```javascript
alert(1)
```

---

# ⚠ Why eval() is Dangerous

If attacker-controlled input reaches `eval()`:
- arbitrary JavaScript execution becomes possible
- attackers can inject malicious code directly

Because of this:
- `eval()` is considered highly dangerous
- secure applications avoid using it completely

---

# 🎯 Objective

The goal of this lab was to:
- identify the vulnerable JavaScript functionality
- break the JSON string safely
- inject arbitrary JavaScript into the `eval()` function
- execute JavaScript successfully

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(1).png?raw=true)

The application initially displayed a normal webpage containing a search functionality.

The word:

```text
hello
```

was entered into the search field for testing.

---

## Screenshot 2 — Reflection Analysis

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(2).png?raw=true)

While inspecting the webpage:
- the word `hello` reflected in the heading

Example:

```html
<h1>0 search result for 'hello'</h1>
```

However:
- nothing immediately vulnerable appeared inside the HTML

This indicated:
- the vulnerability likely existed inside client-side JavaScript

---

## Screenshot 3 — Discovering JavaScript File

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(3).png?raw=true)

A suspicious JavaScript file was discovered:

```html
<script src="/resources/js/searchResults.js"></script>
```

This file was responsible for handling search functionality.

---

## Screenshot 4 — Identifying the Vulnerable eval() Function

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(4).png?raw=true)

After opening the JavaScript file, the following vulnerable code was identified:

```javascript
function search(path) {
    var xhr = new XMLHttpRequest();

    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {

            eval('var searchResultsObj = ' + this.responseText);

            displaySearchResults(searchResultsObj);
        }
    };

    xhr.open("GET", path + window.location.search);
    xhr.send();
};
```

---

# 🔍 Vulnerability Analysis

The dangerous behavior was:

```javascript
eval('var searchResultsObj = ' + this.responseText);
```

The server response:
- was directly concatenated into `eval()`
- became executable JavaScript

This meant:
- breaking the JSON structure could allow arbitrary JavaScript execution

---

# 🖼 Screenshot 5 — Capturing Search Response

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(5).png?raw=true)

The search request was captured:

```http
GET /?search-results?=search=hello
```

The server returned JSON data:

```json
{
  "results":[],
  "searchTerms":"hello"
}
```

This confirmed:
- user-controlled input was reflected inside JSON
- the JSON response was later executed using `eval()`

---

## Screenshot 6 — Sending Request to Repeater

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(6).png?raw=true)

The request was sent to Burp Repeater for manual payload testing.

Purpose:
- understand how escaping behaved
- determine how to break the JSON string safely

---

## Screenshot 7 — Testing Escape Character

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(7).png?raw=true)

Payload tested:

```text
\hello
```

Result:

```json
"searchTerm":"\hello"
```

---

# 🔍 Understanding Backslash Escaping

The backslash (`\`) acts as an escape character in JavaScript strings.

It tells JavaScript:
- treat the next character specially

Example:

```javascript
\" 
```

This escapes a quotation mark.

---

## Screenshot 8 — Testing Double Backslash

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20DOM%20XSS/screenshots/lab10(8).png?raw=true)

Payload tested:

```text
\\hello
```

Observation:
- first backslash escaped the second backslash
- the string remained valid
- JSON structure did not break

This meant:
- additional escaping behavior needed to be bypassed

---

## Screenshot 9 — Breaking the JSON String

![Screenshot 9](screenshot-xss9.png)

Payload tested:

```text
\"hello
```

Result:

```json
{
  "results":[],
  "searchTerm":"\\"hello"
}
```

---

# 🔍 Why the String Broke

What happened:
- the first `\` escaped the second `\`
- the quotation mark (`"`) was no longer escaped correctly
- the JSON string terminated unexpectedly

This successfully broke the JavaScript structure.

---

## Screenshot 10 — Injecting JavaScript

![Screenshot 10](screenshot-xss10.png)

A payload was crafted to:
- break the JSON structure
- execute JavaScript
- safely comment remaining characters

Payload:

```text
\"hello};alert(1);//
```

Result:

```json
{
  "results":[],
  "searchTerm":"\\"hello
};alert(1);//"
}
```

---

# 🔍 Payload Breakdown

## `\"`

Breaks out of the JSON string.

---

## `};`

Closes the JavaScript object safely.

---

## `alert(1);`

Executes arbitrary JavaScript.

---

## `//`

Comments out remaining characters to prevent syntax errors.

---

## Screenshot 11 — Final Payload

![Screenshot 11](screenshot-xss11.png)

The final payload used in the search bar was:

```text
\"};alert(1);//
```

This payload:
- terminated the JSON string
- escaped the JavaScript structure
- injected arbitrary JavaScript successfully

---

## Screenshot 12 — Successful Reflected DOM XSS

![Screenshot 12](screenshot-xss12.png)

After submitting the payload:
- JavaScript executed successfully
- an alert popup appeared
- the vulnerability was confirmed

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- attacker-controlled input was reflected inside JSON
- frontend JavaScript used `eval()` on untrusted data
- proper sanitization and parsing were missing

Example vulnerable behavior:

```javascript
eval(userControlledData)
```

This effectively allows attackers to execute arbitrary JavaScript.

---

# 💥 Impact of Reflected DOM XSS

An attacker can potentially:
- steal session cookies
- hijack accounts
- manipulate webpage content
- execute arbitrary JavaScript
- redirect users to malicious websites
- perform phishing attacks

DOM XSS is especially dangerous because:
- it occurs entirely inside the browser
- many scanners fail to detect it

---

# 🛡 Mitigation

To prevent DOM XSS:
- never use `eval()` on untrusted input
- parse JSON safely using:

```javascript
JSON.parse()
```

instead of:

```javascript
eval()
```

- sanitize user-controlled data
- implement Content Security Policy (CSP)
- avoid dangerous JavaScript execution functions

---

# 🧠 Skills Learned

- Understanding Reflected DOM XSS
- JavaScript escaping behavior
- JSON injection techniques
- Dangerous use of `eval()`
- Breaking JavaScript strings
- Payload crafting
- Client-side JavaScript analysis
- DOM-based exploitation methodology

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how unsafe use of `eval()` with attacker-controlled JSON data can lead to Reflected DOM-Based Cross-Site Scripting vulnerabilities.

By analyzing frontend JavaScript behavior and understanding escape character handling, it was possible to:
- break the JSON structure
- inject arbitrary JavaScript
- execute malicious code successfully

Through this lab, I learned:
- how dangerous `eval()` is
- how JavaScript escaping works
- how JSON injection vulnerabilities occur
- how attackers break JavaScript structures
- how DOM-based vulnerabilities differ from server-side XSS

The lab was successfully exploited by breaking the JSON string and injecting arbitrary JavaScript into the vulnerable `eval()` function.
