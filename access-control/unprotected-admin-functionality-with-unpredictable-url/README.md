# Unprotected Admin Functionality with Unpredictable URL

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | Unprotected admin functionality with unpredictable URL |
| Difficulty | Apprentice |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where sensitive administrative functionality is hidden using an unpredictable URL instead of being protected with proper authorization controls.
While analyzing the application's source code, a hidden administrative endpoint was discovered. Accessing the endpoint directly exposed the administration panel without requiring administrative privileges.
The vulnerability allowed unauthorized deletion of user accounts.

---

# Vulnerability Type

- Broken Access Control
- Unprotected Administrative Interface
- Security Through Obscurity

---

# Impact

An attacker can gain unauthorized access to administrative functionality if hidden endpoints are exposed through client-side code.

This may lead to:
- unauthorized administrative access
- account manipulation
- privilege escalation
- sensitive data exposure
- full application compromise

Using hidden or unpredictable URLs without authorization checks does not provide real security.

---

# Steps to Reproduce

1. Open the target application.
2. Access the browser page source using:
   
```text
View Page Source
```

3. Analyze the source code carefully.
4. Observe that a hidden administrative endpoint is disclosed inside the source code:

```text
/admin-g14cxw
```

5. Access the endpoint directly in the browser.
6. Observe that the administration panel becomes accessible without authorization checks.
7. Delete the user account named `carlos`.
8. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal website without visible administrative functionality.

![Main Website](screenshots/unprotected-admin-unpredictable-1.png)

**Caption:** Initial application interface with no visible admin access.

---

## Step 2 — Discovering Hidden Admin URL in Source Code

While inspecting the page source, a hidden administrative endpoint was discovered.

```text
/admin-g14cxw
```

![image alt](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/unprotected-admin-functionality-with-unpredictable-url/screenshots/(Access%20Control%20Vulnerability)Lab2.png?raw=true)

**Caption:** Hidden administrative URL exposed inside the page source.

---

## Step 3 — Unauthorized Access to Administration Panel

The hidden endpoint was accessed directly, exposing administrative functionality including user management features.

The user `carlos` was deleted successfully, which solved the lab.

![image alt](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/unprotected-admin-functionality-with-unpredictable-url/screenshots/(Access%20Control%20Vulnerability)Lab3.png?raw=true)

**Caption:** Unauthorized access to the administration panel allowing deletion of user accounts.

---

# Why It Works

The application attempted to secure the administration panel by hiding it behind an unpredictable URL instead of implementing proper authorization controls.

Although the URL was not publicly visible in the application's navigation, it remained exposed inside the client-side source code.

Since the server failed to verify user privileges before granting access, any user who discovered the endpoint could directly access the administrative functionality.

This is an example of relying on "security through obscurity," which is not an effective security mechanism.

---

# Security Risk

This vulnerability can result in:
- unauthorized administrative access
- privilege escalation
- account deletion
- sensitive information disclosure
- full application compromise

Hidden URLs should never be considered a replacement for proper authentication and authorization mechanisms.

---

# Recommended Mitigation

- Enforce strict server-side authorization checks.
- Restrict administrative functionality based on user roles.
- Avoid exposing sensitive endpoints in client-side source code.
- Implement proper role-based access control (RBAC).
- Regularly audit hidden routes and administrative functionality.

---

# Learning Section

## What I Learned

- Hidden or unpredictable URLs do not provide real security.
- Client-side source code can expose sensitive application routes.
- Administrative functionality must always be protected server-side.
- Security through obscurity is not a reliable defense mechanism.
- Broken Access Control vulnerabilities can lead to severe security risks.

---

# Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by an unprotected administrative interface hidden behind an unpredictable URL.
By analyzing the application's source code, a hidden administrative endpoint was discovered. Since the application failed to enforce proper authorization checks, the administration panel became accessible to unauthorized users, allowing deletion of user accounts.
