# Method-Based Access Control Can Be Circumvented

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | Method-based access control can be circumvented |
| Difficulty |  PRACTITIONER |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where authorization checks depend only on the HTTP request method.
The administrator account had access to functionality that allowed upgrading or downgrading user privileges through the `/admin-roles` endpoint.
After capturing the administrative request, the session cookie was replaced with a normal user's session cookie. Initially, the request returned an `Unauthorized` response.
However, after changing the HTTP request method, the server processed the request successfully and granted administrative privileges to the standard user account.

---

# Vulnerability Type

- Broken Access Control
- Method-Based Access Control Bypass
- Privilege Escalation
- Improper Authorization Validation

---

# Impact

An attacker can bypass authorization restrictions simply by changing the HTTP request method.

This may lead to:
- privilege escalation
- unauthorized administrative access
- account manipulation
- sensitive functionality exposure
- complete application compromise

Applications should never rely solely on HTTP methods for authorization decisions.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the login page.
3. Login using the administrator credentials:

```text
Username: administrator
Password: admin
```

4. Access the administration panel.
5. Observe the functionality for upgrading or downgrading user privileges.
6. Capture the privilege modification request sent to:

```text
/admin-roles
```

7. Logout from the administrator account.
8. Login using the standard user credentials:

```text
Username: wiener
Password: peter
```

9. Capture the session cookie associated with the `wiener` account.
10. Replace the administrator session cookie in the previously captured `/admin-roles` request with Wiener's session cookie.
11. Send the request.
12. Observe that the server returns an:

```text
Unauthorized
```

response.

13. Change the HTTP request method.
14. Forward the modified request again.
15. Observe that Wiener's account gets promoted to administrator.
16. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal website.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc1.png?raw=true)

**Caption:** Initial application interface before authentication.

---

## Step 2 — Logging in as Administrator

The provided administrator credentials were used to authenticate.

```text
Username: administrator
Password: admin
```

![Administrator Login](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc3.png?raw=true)

**Caption:** Logging into the administrator account.

---

## Step 3 — Accessing Administrative Functionality

After successful authentication, the administration panel became accessible.

![Admin Panel Access](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc4.png?raw=true)

**Caption:** Administrative panel visible after administrator login.

---

## Step 4 — Viewing User Role Management

The administrator account had functionality to upgrade or downgrade user privileges.

![Privilege Management](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc.png?raw=true)

**Caption:** Administrative functionality for modifying user roles.

---

## Step 5 — Capturing the Administrative Request

The privilege modification request was intercepted using Burp Suite.

The request targeted the following endpoint:

```text
/admin-roles
```

![Captured Request](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc5.png?raw=true)

**Caption:** Captured administrative request responsible for modifying user roles.

---

## Step 6 — Logging in as a Standard User

The administrator account was logged out, and the standard user credentials were used:

```text
Username: wiener
Password: peter
```

![Wiener Login](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc6.png?raw=true)

**Caption:** Logging into the application using standard user credentials.

---

## Step 7 — Capturing Wiener's Session Cookie

The login request for the Wiener account was captured, exposing the session cookie.

![Session Cookie](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc7.png?raw=true)

**Caption:** Captured session cookie associated with the Wiener account.

---

## Step 8 — Replacing the Session Cookie

The administrator session cookie in the previously captured `/admin-roles` request was replaced with Wiener's session cookie.

Initially, the server returned an `Unauthorized` response.

![Unauthorized Response](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/Method-Based%20Access%20Control%20Can%20Be%20Circumvented/screenshots/(Access%20Control%20Vulnerability)poc8.png?raw=true)

**Caption:** Authorization failure after replacing the administrator session cookie.

---

## Step 9 — Changing the HTTP Request Method

After changing the HTTP request method, the server processed the request successfully and promoted Wiener to administrator privileges.

The lab was solved successfully.

![Privilege Escalation Successful](screenshots/method-access-control-9.png)

**Caption:** Successful privilege escalation after changing the HTTP request method.

---

# Why It Works

The application implemented inconsistent authorization checks based on the HTTP request method.
While one request method correctly enforced access restrictions, another method processed the same action without proper validation.
By changing the request method, the attacker bypassed the intended access control mechanism and successfully escalated privileges.
This demonstrates why authorization logic should remain consistent across all HTTP methods.

---

# Security Risk

This vulnerability can result in:
- unauthorized privilege escalation
- administrative account compromise
- account manipulation
- sensitive functionality abuse
- full application compromise

Improper method-based authorization logic can allow attackers to bypass critical security controls.

---

# Recommended Mitigation

- Enforce consistent authorization checks across all HTTP methods.
- Validate user privileges server-side for every sensitive request.
- Restrict sensitive functionality to authorized users only.
- Avoid relying on request methods as a security mechanism.
- Perform regular access control testing on all endpoints.

---

# Learning Section

## What I Learned

- Access control logic must remain consistent across HTTP methods.
- Changing request methods can sometimes bypass security restrictions.
- Session cookies can be reused during authorization testing.
- Burp Suite is useful for replaying and modifying authenticated requests.
- Authorization validation should always happen server-side.

---

# Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by inconsistent authorization enforcement across HTTP methods.
Although the application initially blocked the modified request, changing the HTTP method bypassed the restriction and allowed unauthorized privilege escalation, ultimately promoting a standard user account to administrator privileges.
