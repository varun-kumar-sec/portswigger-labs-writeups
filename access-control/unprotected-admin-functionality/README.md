# Unprotected Admin Functionality

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | Unprotected admin functionality |
| Difficulty | Apprentice |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where sensitive administrative functionality is exposed without proper authorization checks.

By accessing the `robots.txt` file, a hidden administrative endpoint was discovered. Visiting the endpoint directly provided unauthorized access to the administration panel, where user management functionality was available.

The vulnerability allowed deletion of user accounts without requiring administrative privileges.

---

# Vulnerability Type

- Broken Access Control
- Unprotected Administrative Interface
- Forced Browsing

---

# Impact

An attacker can gain unauthorized access to sensitive administrative functionality.

This may lead to:
- unauthorized account deletion
- privilege abuse
- exposure of sensitive user data
- administrative compromise

Broken Access Control vulnerabilities are among the most critical web application security issues.

---

# Steps to Reproduce

1. Open the target application.
2. Explore the website manually.
3. Access the following file:

```text
/robots.txt
```

4. Observe that the file discloses a hidden administrative path:

```text
/administrator-panel
```

5. Navigate directly to the disclosed endpoint.
6. Access the administration panel without authentication or authorization checks.
7. Delete the user account named `carlos`.
8. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal web page without any visible administrative functionality.

![image alt](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/unprotected-admin-functionality/screenshots/(Access%20Control%20Vulnerability)Lab1.png?raw=true)

**Caption:** Initial application interface without visible admin access.

---

## Step 2 — Discovering Hidden Admin Endpoint

The `robots.txt` file was accessed manually, revealing a hidden administrative endpoint.

```text
/administrator-panel
```

![image alt](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/unprotected-admin-functionality/screenshots/(Access%20Control%20Vulnerability)Lab2.png?raw=true)

**Caption:** robots.txt exposing the hidden administrative panel URL.

---

## Step 3 — Unauthorized Access to Administration Panel

The hidden endpoint was accessed directly, exposing administrative functionality including user management features.

The user `carlos` was deleted successfully, which solved the lab.

![image alt](screenshots/unprotected-admin-3.png)

**Caption:** Unauthorized access to the administration panel allowing deletion of user accounts.

---

# Why It Works

The application failed to implement proper authorization controls on the administrative endpoint.
Although the admin panel URL was hidden from normal navigation, it remained publicly accessible. Additionally, the `robots.txt` file unintentionally disclosed the sensitive endpoint.
Because the server did not verify whether the user had administrative privileges, any user could directly access the admin functionality.

---

# Security Risk

This vulnerability can lead to:
- unauthorized administrative access
- privilege escalation
- account manipulation
- data exposure
- complete application compromise

Relying on hidden URLs instead of proper authorization checks is not a secure defense mechanism.

---

# Recommended Mitigation

- Implement strict server-side authorization checks.
- Restrict access to administrative functionality based on user roles.
- Do not expose sensitive endpoints in `robots.txt`.
- Use role-based access control (RBAC).
- Regularly audit administrative routes and permissions.

---

# Learning Section

## What I Learned

- Hidden URLs do not provide real security.
- Sensitive endpoints should always require proper authorization.
- `robots.txt` files can accidentally expose important application paths.
- Broken Access Control vulnerabilities are extremely dangerous.
- Administrative functionality must always be protected server-side.

---

# Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by an unprotected administrative interface.
By accessing the `robots.txt` file, a hidden administration endpoint was discovered. Since the application failed to enforce authorization checks, the admin panel became publicly accessible, allowing unauthorized deletion of user accounts.
