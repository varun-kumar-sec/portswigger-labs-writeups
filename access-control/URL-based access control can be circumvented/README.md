# URL-Based Access Control Can Be Circumvented

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | URL-based access control can be circumvented |
| Difficulty | Apprentice |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where access restrictions can be bypassed using the `X-Original-URL` HTTP header.

Although direct access to the administrative panel returned an "Access Denied" response, the backend server trusted the `X-Original-URL` header and processed the request as an internal administrative request.

By modifying HTTP requests in Burp Suite and inserting the following header:

```http
X-Original-URL: /admin
```

administrative functionality became accessible.

The vulnerability allowed unauthorized deletion of user accounts.

---

# Vulnerability Type

- Broken Access Control
- URL-Based Access Control Bypass
- Header Manipulation
- Reverse Proxy Misconfiguration

---

# Impact

An attacker can bypass access restrictions by manipulating internal routing headers.

This may lead to:
- unauthorized administrative access
- privilege escalation
- account manipulation
- sensitive data exposure
- complete application compromise

Improper trust in user-controlled headers can create severe security vulnerabilities.

---

# Steps to Reproduce

1. Open the target application.
2. Observe that an **Admin Panel** button is visible.
3. Attempt to access the admin panel directly.
4. Observe that access is denied.
5. Capture the admin panel request using Burp Suite.
6. Modify the request by adding the following header:

```http
X-Original-URL: /admin
```

7. Forward the modified request.
8. Observe that the response changes from:

```http
403 Forbidden
```

to:

```http
200 OK
```

9. Observe that administrative functionality becomes accessible, including:

```text
/admin/delete/?username=carlos
```

10. Modify the request as follows:

```http
POST /?username=carlos HTTP/1.1
X-Original-URL: /admin/delete
```

11. Forward the request.
12. The user account `carlos` gets deleted successfully.
13. The lab gets solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially displayed a normal web page with an additional **Admin Panel** button.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/URL-based%20access%20control%20can%20be%20circumvented/screenshots/(Access%20Control%20Vulnerability)poc1.png?raw=true)

**Caption:** Initial application interface displaying the admin panel option.

---

## Step 2 — Access Denied on Direct Access

Attempting to access the admin panel directly resulted in an access denied response.

![Access Denied](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/URL-based%20access%20control%20can%20be%20circumvented/screenshots/(Access%20Control%20Vulnerability)poc2.png?raw=true)

**Caption:** Direct access to the admin panel blocked by the application.

---

## Step 3 — Modifying the Request Header

The admin panel request was intercepted using Burp Suite.

The following header was added:

```http
X-Original-URL: /admin
```

![Modified Header](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/URL-based%20access%20control%20can%20be%20circumvented/screenshots/(Access%20Control%20Vulnerability)poc3.png?raw=true)

**Caption:** Injecting the `X-Original-URL` header to bypass access control.

---

## Step 4 — Accessing Administrative Functionality

After inserting the header, the server returned a:

```http
200 OK
```

response, exposing administrative functionality including account deletion features.

![Admin Access Granted](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/URL-based%20access%20control%20can%20be%20circumvented/screenshots/(Access%20Control%20Vulnerability)poc4.png?raw=true)

**Caption:** Unauthorized administrative access after header manipulation.

---

## Step 5 — Deleting Carlos's Account

The request was modified as follows:

```http
POST /?username=carlos HTTP/1.1
X-Original-URL: /admin/delete
```

This successfully deleted Carlos's account and solved the lab.

![Lab Solved](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/URL-based%20access%20control%20can%20be%20circumvented/screenshots/(Access%20Control%20Vulnerability)poc5.png?raw=true)

**Caption:** Successfully deleting Carlos's account using the manipulated request.

---

# Why It Works

The application attempted to restrict administrative functionality based on URL paths.
However, the backend server trusted the `X-Original-URL` header and used it to determine the actual requested endpoint.
Because the server failed to validate whether the header originated from a trusted internal source, attackers could manipulate it directly.
This allowed unauthorized users to bypass frontend access restrictions and access administrative functionality.

---

# Security Risk

This vulnerability can result in:
- access control bypass
- unauthorized administrative access
- privilege escalation
- account deletion
- sensitive data exposure

Applications that trust user-controlled routing headers are vulnerable to serious backend security bypasses.

---

# Recommended Mitigation

- Do not trust client-controlled routing headers.
- Enforce authorization checks on the backend server.
- Restrict internal headers to trusted reverse proxies only.
- Validate all administrative requests server-side.
- Disable unnecessary rewrite or routing headers.

---

# Learning Section

## What I Learned

- Access control restrictions can sometimes be bypassed using HTTP headers.
- The `X-Original-URL` header may influence backend routing behavior.
- Frontend access restrictions alone are not sufficient for security.
- Burp Suite is useful for manipulating HTTP requests and headers.
- Proper server-side authorization checks are critical.

---

# Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by improper trust in the `X-Original-URL` header.
Although direct access to the administration panel was blocked, modifying the request header allowed unauthorized access to backend administrative functionality, ultimately leading to successful deletion of a user account.
