# User Role Controlled by Request Parameter

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | User role controlled by request parameter |
| Difficulty | Apprentice |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where administrative privileges are controlled through a client-side request parameter.
After logging into the application using normal user credentials, the login request contained a parameter named:

```text
Admin=false
```

By modifying the parameter to:

```text
Admin=true
```

using Burp Suite's Match and Replace feature, administrative functionality became accessible without proper authorization checks.
The vulnerability allowed unauthorized access to the administration panel and deletion of user accounts.

---

# Vulnerability Type

- Broken Access Control
- Privilege Escalation
- Parameter Tampering
- Client-Side Authorization Control

---

# Impact

An attacker can manipulate request parameters to gain elevated privileges within the application.

This may lead to:
- unauthorized administrative access
- account deletion
- privilege escalation
- sensitive data exposure
- full application compromise

Applications should never rely on client-controlled parameters to determine user privileges.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the login page.
3. Login using the provided credentials:

```text
Username: wiener
Password: peter
```

4. Capture the login request using Burp Suite.
5. Observe the following parameter inside the request:

```text
Admin=false
```

6. Configure Burp Suite Match and Replace feature to replace:

```text
Admin=false
```

with:

```text
Admin=true
```

7. Refresh the logged-in page.
8. Observe that administrative functionality becomes available.
9. Access the administration panel.
10. Delete the user account named `carlos`.
11. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal website.

![image alt](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/User%20role%20controlled%20by%20request%20parameter/screenshots/(Access%20Control%20Vulnerability)poc1.png?raw=true)

**Caption:** Initial application interface before authentication.

---

## Step 2 — Logging into the Application

The provided credentials were used to authenticate as a normal user.

```text
Username: wiener
Password: peter
```

![image alt](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/User%20role%20controlled%20by%20request%20parameter/screenshots/(Access%20Control%20Vulnerability)poc2.png?raw=true)

**Caption:** Logging into the application using standard user credentials.

---

## Step 3 — Capturing the Login Request

The login request was intercepted using Burp Suite.

Inside the request, the following parameter was identified:

```text
Admin=false
```

![Captured Request](screenshots/user-role-request-3.png)

**Caption:** Login request containing the `Admin=false` parameter.

---

## Step 4 — Modifying the Request Parameter

Burp Suite Match and Replace feature was configured to automatically replace:

```text
Admin=false
```

with:

```text
Admin=true
```

for all upcoming requests.

![Match and Replace Configuration](screenshots/user-role-request-4.png)

**Caption:** Burp Suite Match and Replace configured to elevate privileges.

---

## Step 5 — Accessing Administrative Functionality

After refreshing the page, an administrative panel button became visible for the standard user account.

![Admin Button Visible](screenshots/user-role-request-5.png)

**Caption:** Administrative functionality exposed after modifying the request parameter.

---

## Step 6 — Unauthorized Access to Admin Panel

The administration panel became accessible, allowing management of user accounts.

The user `carlos` was deleted successfully, which solved the lab.

![Admin Panel Access](screenshots/user-role-request-6.png)

**Caption:** Unauthorized access to the administration panel allowing deletion of user accounts.

---

# Why It Works

The application relied on a client-controlled parameter to determine whether the user had administrative privileges.

Because the parameter was sent from the client side, it could easily be modified using Burp Suite.

The server trusted the manipulated value without performing proper authorization validation, allowing privilege escalation.

This demonstrates why authorization decisions should never rely on user-controlled input.

---

# Security Risk

This vulnerability can result in:
- privilege escalation
- unauthorized administrative access
- account manipulation
- sensitive data exposure
- complete application compromise

Attackers can modify client-side parameters to gain unauthorized functionality if proper server-side validation is missing.

---

# Recommended Mitigation

- Never trust client-controlled parameters for authorization decisions.
- Enforce strict server-side role validation.
- Implement secure session management.
- Use role-based access control (RBAC).
- Validate user privileges on every sensitive request.

---

# Learning Section

## What I Learned

- Client-side parameters should never determine user privileges.
- Burp Suite Match and Replace can be used to manipulate application behavior.
- Authorization checks must always happen server-side.
- Parameter tampering can lead to privilege escalation.
- Broken Access Control vulnerabilities are highly critical.

---

# Tools Used

- Burp Suite
- Burp Suite Match and Replace
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by insecure privilege management using a client-controlled request parameter.
By modifying the `Admin=false` parameter to `Admin=true`, administrative functionality became accessible without proper authorization checks, allowing unauthorized deletion of user accounts.
