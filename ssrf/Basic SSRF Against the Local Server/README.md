# Basic SSRF Against the Local Server

## 📌 Lab Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability.

The application contained a stock-checking functionality that allowed the backend server to fetch data from a URL specified by the user. Because the application failed to properly validate the destination URL, it became possible to force the server to make requests to internal resources that were not directly accessible from the internet.

The vulnerability existed because:

- user-controlled input was used to construct backend requests
- the application trusted arbitrary URLs supplied by the user
- internal resources were accessible from the backend server
- no restrictions were placed on requests to localhost

This lab focused on:

- Server-Side Request Forgery (SSRF)
- Internal Network Access
- Backend Request Manipulation
- Localhost Enumeration
- Administrative Interface Exposure

---

# 🔍 What is SSRF?

**Server-Side Request Forgery (SSRF)** occurs when an attacker can force a server to make HTTP requests on their behalf.

Normally:

```text
User
 ↓
Website
 ↓
External Service
```

The website decides where requests are sent.

However, in SSRF:

```text
Attacker
 ↓
Website
 ↓
Internal Server
```

the attacker controls the destination.

As a result, the attacker can access resources that are normally unreachable from the internet, such as:

```text
localhost
127.0.0.1
Internal APIs
Admin Panels
Cloud Metadata Services
```

Since the request originates from the trusted backend server, internal systems often allow access.

---

# ⚠ Why Was the Application Vulnerable?

The stock-checking functionality accepted a URL parameter:

```text
stockApi=<URL>
```

The backend server then made a request to that URL and returned the response.

Instead of restricting requests to the legitimate stock API server, the application allowed arbitrary destinations.

This meant an attacker could replace:

```text
http://stock.weliketoshop.net
```

with:

```text
http://localhost
```

and force the backend to access internal services.

---

# 🎯 Objective

The goal of this lab was to:

- identify the SSRF entry point
- access the internal admin panel
- discover administrative functionality
- delete the Carlos user
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20the%20Local%20Server/screenshots/lab1(1).png?raw=true)

The application initially displayed:

- various products
- a **View Details** button for each product

At this stage, there was no visible indication of an SSRF vulnerability.

---

## Screenshot 2 — Product Details Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20the%20Local%20Server/screenshots/lab1(2).png?raw=true)

After clicking a product's **View Details** button, I was redirected to the product page.

The page contained a:

```text
Check Stock
```

button.

When clicked, the application returned:

```text
73 units
```

This suggested that the application was retrieving stock information dynamically from another system.

---

## Screenshot 3 — Identifying the SSRF Entry Point

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20the%20Local%20Server/screenshots/lab1(3).png?raw=true)

I captured the stock check request:

```http
POST /product/stock
```

The request contained the parameter:

```text
stockApi=http://stock.weliketoshop.net:8080/product/stock/check?productId=1&storeId=1
```

This was extremely interesting because:

```text
User Input
        ↓
URL
        ↓
Backend Request
```

The backend server was making requests based on the supplied URL.

This became the SSRF attack surface.

---

## Screenshot 4 — Accessing the Internal Admin Panel

![Screenshot 4](screenshot-ssrf4.png)

The lab provided a hint that an admin panel existed on:

```text
http://localhost/admin
```

I replaced the original URL with:

```text
stockApi=http://localhost/admin
```

and sent the request.

The server responded with:

```http
200 OK
```

When rendering the response, I was able to see the internal admin panel.

The panel displayed:

```text
wiener
carlos
```

along with delete functionality for each user.

This confirmed that:

```text
Backend Server
        ↓
Can Access Localhost
        ↓
SSRF Successful
```

---

## Screenshot 5 — Discovering the Delete Functionality

![Screenshot 5](screenshot-ssrf5.png)

After examining the response in the Pretty view, I found a link responsible for deleting Carlos:

```html
<a href="/admin/delete?username=carlos">
```

This revealed the internal administrative endpoint:

```text
/admin/delete?username=carlos
```

I copied this path for the next step.

---

## Screenshot 6 — Deleting Carlos

![Screenshot 6](screenshot-ssrf6.png)

I modified the parameter again:

```text
stockApi=http://localhost/admin/delete?username=carlos
```

and sent the request.

The backend server performed the request on my behalf.

As a result:

```text
Carlos Account Deleted
```

and the lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application trusted user-supplied URLs and used them directly in backend requests.

The workflow looked like:

```text
User Input
        ↓
stockApi Parameter
        ↓
Backend Request
        ↓
Response Returned
```

Without proper validation, an attacker could redirect requests to:

```text
localhost
127.0.0.1
Internal APIs
Administrative Interfaces
```

and gain access to internal resources.

---

# 💥 Impact

An attacker could potentially:

- access internal administrative interfaces
- interact with internal APIs
- bypass network restrictions
- enumerate internal services
- access cloud metadata endpoints
- gain sensitive information
- perform unauthorized administrative actions

---

# 🛡 Mitigation

To prevent SSRF vulnerabilities:

- never trust user-supplied URLs
- implement strict allowlists
- block localhost and internal IP ranges
- restrict outbound requests
- isolate backend services
- validate destination hosts
- use network-level filtering

Example blocked destinations:

```text
127.0.0.1
localhost
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

---

# 🧠 Skills Learned

- Server-Side Request Forgery (SSRF)
- Backend Request Analysis
- Internal Resource Discovery
- Localhost Enumeration
- Administrative Interface Access
- Burp Repeater Testing
- Request Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how a stock-checking feature can become vulnerable when user-controlled URLs are trusted by the backend server.

By identifying the SSRF entry point, replacing the legitimate stock API URL with an internal localhost address, and interacting with the hidden administrative interface, I was able to access internal functionality and delete the Carlos user.

Through this lab, I learned:

- how SSRF vulnerabilities work
- how backend systems make server-side requests
- how internal resources can become exposed
- how localhost services can be accessed through SSRF
- how administrative functionality can be abused through backend request forgery

The lab was successfully solved by exploiting a Server-Side Request Forgery vulnerability to access the internal admin panel and delete the Carlos account.
