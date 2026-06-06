# Blind SSRF with Out-of-Band Detection

## 📌 Lab Overview

This lab demonstrates a **Blind Server-Side Request Forgery (Blind SSRF)** vulnerability.

Unlike traditional SSRF, the application does not return the response from the server-side request back to the attacker. As a result, attackers cannot directly see the result of their injected request.

To detect the vulnerability, an **Out-of-Band (OAST)** technique is used. By supplying a URL under the attacker's control, it becomes possible to observe whether the vulnerable server makes an outbound request.

This lab focused on:

- Blind SSRF
- Out-of-Band (OAST) detection
- HTTP header injection
- Burp Collaborator
- Backend request verification

---

# 🌐 What is SSRF?

**Server-Side Request Forgery (SSRF)** occurs when a web application fetches a URL supplied by a user without properly validating it.

Instead of the server requesting trusted resources:

```text
User → Web Server → Trusted API
```

an attacker can force it to request:

```text
User → Web Server → Internal Systems
```

or

```text
User → Web Server → Attacker Server
```

This allows attackers to:

- access internal services
- scan internal networks
- interact with cloud metadata services
- bypass firewalls
- perform internal reconnaissance

---

# 📡 What is an Out-of-Band (OAST) Technique?

In many SSRF vulnerabilities, the application never displays the server's response.

Example:

```text
Attacker → SSRF Payload
Server → Internal Request
```

But the attacker sees only:

```text
Success
```

or

```text
Error
```

without any response data.

This is known as:

```text
Blind SSRF
```

To verify whether the request actually occurred, attackers use an **Out-of-Band Application Security Testing (OAST)** technique.

Instead of targeting an internal server:

```text
http://internal-server/admin
```

they target a server they control:

```text
http://attacker-server.com
```

If the vulnerable server connects back:

```text
Victim Server → Attacker Server
```

the attacker can observe the interaction.

Burp Suite provides this functionality through:

```text
Burp Collaborator
```

---

# 🎯 Objective

The goal of this lab was to:

- identify an SSRF injection point
- trigger a server-side request
- verify the request using Burp Collaborator
- prove the existence of Blind SSRF

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-ssrf1.png)

The application displayed:

- various products
- a **View Details** button for each product

At this stage, no obvious SSRF functionality was visible.

---

## Screenshot 2 — Viewing a Product

![Screenshot 2](screenshot-ssrf2.png)

I clicked a random product's:

```text
View Details
```

button.

The page displayed:

- product information
- product description

Nothing immediately appeared vulnerable.

---

## Screenshot 3 — Capturing the Request

![Screenshot 3](screenshot-ssrf3.png)

I captured the request:

```http
GET /product?productId=1
```

The response returned:

```http
200 OK
```

While inspecting the request, I noticed an interesting header:

```http
Referer: https://0aa3001a044efad6811a849900300075.web-security-academy.net/
```

### 🔍 What is the Referer Header?

The **Referer** header tells a server where the request originated from.

Example:

```http
GET /product?productId=1 HTTP/1.1
Referer: https://example.com/products
```

Meaning:

```text
Current Request
      ↑
Came From
      ↑
https://example.com/products
```

Applications commonly use it for:

- analytics
- logging
- tracking user navigation
- access control
- security monitoring

In this lab, the backend was processing the Referer header, making it a potential SSRF injection point.

---

## Screenshot 4 — Injecting Burp Collaborator

![Screenshot 4](screenshot-ssrf4.png)

I replaced the Referer header value with my Burp Collaborator URL:

```http
Referer: https://rje081pfdmbapul5uf7v9txad1js7iv7.oastify.com/
```

Then I sent the request.

The objective was:

```text
Application
      ↓
Processes Referer
      ↓
Makes Request To Collaborator
```

If the application attempted to access the supplied URL, the interaction would appear in Burp Collaborator.

---

## Screenshot 5 — Receiving Out-of-Band Interaction

![Screenshot 5](screenshot-ssrf5.png)

After checking Burp Collaborator, I observed:

```text
2 DNS Interactions
1 HTTP Interaction
```

This confirmed that the vulnerable server had successfully made requests to my Collaborator domain.

The flow looked like:

```text
Attacker
    ↓
Inject Collaborator URL
    ↓
Application Backend
    ↓
DNS Lookup
    ↓
HTTP Request
    ↓
Collaborator Server
```

Because the interaction occurred outside the application, it proved the existence of a **Blind SSRF vulnerability**.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application trusted user-controlled input from the:

```http
Referer
```

header.

The backend then performed a request to the supplied URL without validation.

Example:

```text
User Input
     ↓
Backend Request
     ↓
External Resource Access
```

This behavior allowed attackers to force the server into making arbitrary outbound requests.

---

# 💥 Impact

A Blind SSRF vulnerability can allow attackers to:

- scan internal networks
- identify hidden services
- access internal applications
- interact with cloud metadata endpoints
- bypass firewall restrictions
- perform reconnaissance against backend infrastructure

In real-world environments, SSRF can often lead to full server compromise.

---

# 🛡 Mitigation

To prevent Blind SSRF vulnerabilities:

- never trust user-supplied URLs
- validate and whitelist allowed destinations
- block requests to internal IP ranges
- restrict outbound network access
- disable unnecessary URL fetching functionality
- monitor unexpected outbound requests
- implement network segmentation

---

# 🧠 Skills Learned

- SSRF Testing
- Blind SSRF Detection
- Out-of-Band (OAST) Techniques
- Burp Collaborator Usage
- HTTP Header Analysis
- Backend Request Mapping
- Referer Header Abuse

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Burp Collaborator
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how a seemingly harmless HTTP header can become an SSRF injection point.

By identifying that the application processed the:

```http
Referer
```

header server-side and replacing it with a Burp Collaborator URL, I was able to trigger an outbound request from the backend server.

Since the application did not return the response directly, Burp Collaborator was used to verify the interaction through DNS and HTTP requests.

Through this lab, I learned:

- how Blind SSRF differs from normal SSRF
- how Out-of-Band (OAST) techniques work
- how Burp Collaborator detects backend interactions
- how HTTP headers can become SSRF attack vectors

The lab was successfully solved by exploiting a Blind SSRF vulnerability and confirming backend interaction through Burp Collaborator.
