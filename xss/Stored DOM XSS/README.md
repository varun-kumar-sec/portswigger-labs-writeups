# Stored DOM XSS

## 📌 Lab Overview

This lab demonstrates a **Stored DOM-Based Cross-Site Scripting (Stored DOM XSS)** vulnerability.

Unlike normal Stored XSS:
- the payload was stored inside the application
- JavaScript on the client side dynamically processed the stored comments
- the vulnerability existed in the browser DOM manipulation logic

This lab focused on:
- browser parsing behavior
- DOM processing
- script tag handling
- bypassing malformed HTML parsing

---

# 🔍 What is Stored DOM XSS?

Stored DOM XSS occurs when:
- attacker-controlled input is permanently stored by the application
- frontend JavaScript later processes that stored content
- unsafe DOM manipulation causes JavaScript execution inside the browser

This combines:
- Stored XSS persistence
- DOM-based client-side execution

---

# 🎯 Objective

The goal of this lab was to inject a payload into the comment system and achieve JavaScript execution inside the browser through DOM processing.

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(1).png?raw=true)

The application initially displayed a normal webpage containing a **View Post** functionality.

At this stage:
- no payload was injected
- application behaved normally

---

## Screenshot 2 — Posting a Normal Comment

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(2).png?raw=true)

A random blog post was opened, revealing a **Leave a Comment** functionality.

The following details were submitted:

### Comment

```text
hello
```

### Name

```text
abcd
```

### Email

```text
abcd@gmail.com
```

### Website

```text
http://google.com
```

This step was performed to observe:
- how comments were rendered
- how the application processed user input

---

## Screenshot 3 — Comment Submission

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(3).png?raw=true)

After submitting the comment:
- the application redirected to a confirmation page

Message displayed:

```text
Thank you for your comment! your comment has been submitted.
```

A **Back to Blog** button was also visible.

---

## Screenshot 4 — Initial XSS Testing

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(4).png?raw=true)

The previously submitted comment became visible with other user comments.

After inspecting the comment:
- nothing immediately vulnerable was identified

A new payload was then tested:

```html
<script>alert(1)</script>
```

Submitted values:

### Comment

```html
<script>alert(1)</script>
```

### Name

```text
abcd
```

### Email

```text
abcd@gmail.com
```

### Website

```text
http://google.com
```

---

## Screenshot 5 — Analyzing Browser Parsing Behavior

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(5).png?raw=true)

After posting the payload:
- the comment appeared malformed
- the closing `</script>` tag was missing visually

Inspection revealed:

```html
<script>loadComments('/post/comment')</script>

<section class="comment">
...
</section>

<section class="comment">
    <p>...</p>
    <p>
        <script>alert(1)
        <script></script>
    </p>
    <p></p>
</section>
```

---

# 🔍 Understanding the Issue

The browser interpreted the injected payload incorrectly because:
- the application already used script tags internally
- the browser attempted to merge nested script elements
- malformed parsing prevented execution

The injected `<script>` became trapped inside another script context.

---

## Screenshot 6 — Testing Another Payload

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(6).png?raw=true)

A modified payload was submitted:

```html
<script><script>alert(1)</script>
```

Purpose:
- manipulate browser parsing behavior
- attempt to create a clean executable script tag

---

## Screenshot 7 — Understanding First Occurrence Parsing

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(7).png?raw=true)

Inspection revealed:

```html
<p><script>alert(1)</p>

<section class="comment">
    <p>
        <script>
        <script>alert(1)</script>
    </p>
</section>
```

---

# 🔍 Why the Payload Still Failed

The browser treated:
- the first `<script>` tag as the primary script context
- subsequent script tags became nested incorrectly

This caused:
- payload corruption
- script execution failure

The first script occurrence essentially blocked later script execution.

---

# 🔍 Important Concept — Browser HTML Parsing

Browsers automatically attempt to repair malformed HTML.

In this case:
- nested `<script>` tags created parsing confusion
- browser merged or ignored parts of the payload
- JavaScript never executed successfully

This is common in DOM-based parsing vulnerabilities.

---

## Screenshot 8 — Final Payload Using img Tag

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(8).png?raw=true)

A new payload was crafted:

```html
<script><img src=x onerror=alert(1)>
```

---

# 🔍 Payload Breakdown

## First `<script>` Tag

```html
<script>
```

This occupied the first script occurrence position.

The browser focused on this script block first.

---

## `<img src=x>`

Creates an invalid image.

Because:
- image source `x` does not exist
- browser fails to load the image

the error handler executes automatically.

---

## `onerror=alert(1)`

Triggers JavaScript execution when image loading fails.

---

# 🔍 Why This Payload Worked

The important difference was:
- only one `<script>` tag existed
- the executable code used an `<img>` tag instead

So:
- browser parsing confusion was avoided
- the image tag executed independently
- `onerror` triggered successfully

---

## Screenshot 9 — Successful Stored DOM XSS

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Stored%20DOM%20XSS/screenshots/lab9(9).png?raw=true)

After loading the malicious comment:
- browser attempted to load the invalid image
- image loading failed
- the `onerror` event executed automatically
- an alert popup appeared

This confirmed successful Stored DOM XSS exploitation.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- attacker-controlled comments were stored permanently
- frontend JavaScript dynamically processed stored content
- dangerous HTML content was inserted into the DOM
- browser parsing behavior was not safely handled

The application trusted unsafe user input inside dynamic DOM rendering.

---

# 💥 Impact of Stored DOM XSS

An attacker can potentially:
- steal session cookies
- hijack accounts
- execute arbitrary JavaScript
- manipulate webpage content
- perform phishing attacks
- attack every user viewing the page

Stored DOM XSS is especially dangerous because:
- payloads persist permanently
- victims trigger the attack automatically

---

# 🛡 Mitigation

To prevent Stored DOM XSS:
- sanitize user-controlled HTML
- avoid dangerous DOM insertion methods
- encode output properly
- use safe rendering APIs
- implement Content Security Policy (CSP)
- avoid inserting raw HTML into DOM structures

---

# 🧠 Skills Learned

- Understanding Stored DOM XSS
- Browser HTML parsing behavior
- Nested script handling
- Event handler exploitation
- DOM rendering analysis
- Malformed HTML behavior
- XSS payload crafting
- Parsing confusion bypass techniques

---

# 🧰 Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how unsafe client-side DOM processing of stored user input can lead to Stored DOM XSS vulnerabilities.

By analyzing:
- browser parsing behavior
- nested script handling
- DOM rendering logic

it was possible to craft a payload that bypassed parsing issues and achieved JavaScript execution successfully.

Through this lab, I learned:
- how browsers process malformed HTML
- why nested script tags behave unpredictably
- how event handlers can bypass parsing restrictions
- how Stored DOM XSS differs from normal Stored XSS
- how frontend DOM processing introduces security risks

The lab was successfully exploited using an image error event handler payload that bypassed browser script parsing limitations and triggered JavaScript execution automatically.
