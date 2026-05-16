# User Role Can Be Modified in User Profile

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | User role can be modified in user profile |
| Difficulty | Apprentice |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where user privilege levels can be modified directly through profile update requests.

After logging into the application using normal user credentials, the email update functionality was analyzed using Burp Suite. While inspecting the server response, a parameter named:

```json
"roleid":1
```

was identified.

By modifying the request and adding:

```json
"roleid":2
```

the application granted administrative privileges to the standard user account.

This allowed unauthorized access to the administration panel and deletion of user accounts.

---

# Vulnerability Type

- Broken Access Control
- Privilege Escalation
- Parameter Tampering
- Insecure User Profile Management

---

# Impact

An attacker can manipulate profile update requests to gain elevated privileges within the application.

This may lead to:
- unauthorized administrative access
- privilege escalation
- account manipulation
- sensitive data exposure
- complete application compromise

Applications should never allow privilege-related parameters to be controlled by users.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the login page.
3. Login using the provided credentials:

```text
Username: wiener
Password: peter
```

4. Access the profile update functionality.
5. Update the email address using:

```text
hello@hello.com
```

6. Capture the update email request using Burp Suite.
7. Observe the following role identifier in the response:

```json
"roleid":1
```

8. Modify the request body and add:

```json
"roleid":2
```

9. Forward the modified request.
10. Refresh the logged-in page.
11. Observe that administrative functionality becomes available.
12. Access the administration panel.
13. Delete the user account named `carlos`.
14. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal website.

![Main Website](screenshots/user-role-profile-1.png)

**Caption:** Initial application interface before authentication.

---

## Step 2 — Logging into the Application

The provided credentials were used to authenticate as a normal user.

```text
Username: wiener
Password: peter
```

![Login Page](screenshots/user-role-profile-2.png)

**Caption:** Logging into the application using standard user credentials.

---

## Step 3 — Accessing the Profile Update Function

After logging in, the profile update functionality became available.

The email address used was:

```text
hello@hello.com
```

![Update Email Function](screenshots/user-role-profile-3.png)

**Caption:** User profile section containing the email update functionality.

---

## Step 4 — Inspecting the Server Response

The update email request was captured using Burp Suite.

Inside the response, an interesting parameter was identified:

```json
"roleid":1
```

![Captured Response](screenshots/user-role-profile-4.png)

**Caption:** Server response exposing the current role identifier.

---

## Step 5 — Modifying the Role Identifier

The request was modified by adding the following JSON parameter:

```json
"roleid":2
```

This attempted to escalate the user's privileges.

![Modified Request](screenshots/user-role-profile-5.png)

**Caption:** Modified profile update request attempting privilege escalation.

---

## Step 6 — Administrative Access Granted

After refreshing the logged-in page, an administrative panel button became visible.

![Admin Button Visible](screenshots/user-role-profile-6.png)

**Caption:** Administrative functionality exposed after modifying the role identifier.

---

## Step 7 — Unauthorized Access to Administration Panel

The administration panel became accessible, allowing management of user accounts.

The user `carlos` was deleted successfully, which solved the lab.

![Admin Panel Access](screenshots/user-role-profile-7.png)

**Caption:** Unauthorized access to the administration panel allowing deletion of user accounts.

---

# Why It Works

The application trusted user-controlled input during the profile update process.

By modifying the request and supplying a higher role identifier, the server updated the user's privileges without verifying whether the action was authorized.

The application failed to implement proper server-side access control validation, allowing privilege escalation through parameter tampering.

Authorization-related values should never be modifiable by normal users.

---

# Security Risk

This vulnerability can result in:
- unauthorized privilege escalation
- administrative account takeover
- sensitive data exposure
- account manipulation
- complete application compromise

If attackers can manipulate role identifiers, they may gain full control over the application.

---

# Recommended Mitigation

- Never trust client-controlled role identifiers.
- Enforce strict server-side authorization checks.
- Prevent users from modifying privilege-related parameters.
- Implement secure role-based access control (RBAC).
- Validate all sensitive actions on the server side.

---

# Learning Section

## What I Learned

- User privilege levels should never be controlled through client-side requests.
- Burp Suite can be used to identify hidden parameters in requests and responses.
- Parameter tampering can lead to privilege escalation.
- Authorization checks must always happen server-side.
- Improper profile management functionality can expose critical security flaws.

---

# Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by insecure handling of privilege-related parameters during profile updates.

By modifying the request and adding `"roleid":2`, administrative privileges were granted to a standard user account. This allowed unauthorized access to the administration panel and deletion of user accounts.
