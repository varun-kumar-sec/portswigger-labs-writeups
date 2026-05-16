# User ID Controlled by Request Parameter

## Lab Information

| Category | Access Control |
|---|---|
| Lab Name | User ID controlled by request parameter |
| Difficulty | Apprentice |
| Platform | PortSwigger Web Security Academy |

---

# Description

This lab demonstrates a Broken Access Control vulnerability where user account access is controlled directly through a URL parameter.

After logging into the application using standard user credentials, the account page URL contained the logged-in username as a request parameter:

```text
/my-account?id=wiener
```

By modifying the parameter value from `wiener` to `carlos`, unauthorized access to another user's account became possible.

This exposed sensitive information including Carlos's API key.

---

# Vulnerability Type

- Broken Access Control
- Insecure Direct Object Reference (IDOR)
- Parameter Tampering

---

# Impact

An attacker can manipulate request parameters to access sensitive information belonging to other users.

This may lead to:
- unauthorized account access
- exposure of sensitive data
- account takeover
- privacy violations
- privilege abuse

Improper validation of user-controlled identifiers can expose confidential information.

---

# Steps to Reproduce

1. Open the target application.
2. Navigate to the login page.
3. Login using the provided credentials:

```text
Username: wiener
Password: peter
```

4. Access the account page after successful authentication.
5. Observe the following URL structure:

```text
/my-account?id=wiener
```

6. Modify the parameter value from:

```text
wiener
```

to:

```text
carlos
```

7. Press Enter and access the modified URL.
8. Observe that Carlos's account page becomes accessible.
9. Copy Carlos's API key.
10. Submit the API key in the "Submit Solution" section.
11. The lab gets successfully solved.

---

# Proof of Concept (PoC)

## Step 1 — Accessing the Main Website

The application initially appears to be a normal website.

![Main Website](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/User%20ID%20controlled%20by%20request%20parameter/screenshots/(Access%20Control%20Vulnerability)poc1.png?raw=true)

**Caption:** Initial application interface before authentication.

---

## Step 2 — Logging into the Application

The provided credentials were used to authenticate as a normal user.

```text
Username: wiener
Password: peter
```

![Login Page](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/User%20ID%20controlled%20by%20request%20parameter/screenshots/(Access%20Control%20Vulnerability)poc2.png?raw=true)

**Caption:** Logging into the application using standard user credentials.

---

## Step 3 — Identifying the User ID Parameter

After logging in, the account page URL contained the username as a request parameter.

```text
/my-account?id=wiener
```

![User ID Parameter](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/User%20ID%20controlled%20by%20request%20parameter/screenshots/(Access%20Control%20Vulnerability)poc3.png?raw=true)

**Caption:** Account page URL exposing the user identifier parameter.

---

## Step 4 — Accessing Another User's Account

The username inside the URL parameter was changed from:

```text
wiener
```

to:

```text
carlos
```

This granted unauthorized access to Carlos's account, exposing his API key.

![Carlos Account Access](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/access-control/User%20ID%20controlled%20by%20request%20parameter/screenshots/(Access%20Control%20Vulnerability)poc4.png?raw=true)

**Caption:** Unauthorized access to Carlos's account after modifying the user ID parameter.

---

## Step 5 — Submitting the API Key

Carlos's API key was submitted in the solution section, successfully solving the lab.

![Lab Solved](screenshots/user-id-request-5.png)

**Caption:** Submitting Carlos's API key to complete the lab.

---

# Why It Works

The application relied entirely on a user-controlled request parameter to determine which account data should be displayed.

The server failed to verify whether the authenticated user was authorized to access the requested account.

Because the parameter could be modified directly in the URL, attackers could access sensitive information belonging to other users.

This is a classic example of an Insecure Direct Object Reference (IDOR) vulnerability.

---

# Security Risk

This vulnerability can result in:
- unauthorized access to user accounts
- exposure of sensitive information
- account takeover
- privacy violations
- privilege abuse

Applications that trust user-controlled identifiers without authorization checks are highly vulnerable to IDOR attacks.

---

# Recommended Mitigation

- Enforce strict server-side authorization checks.
- Validate that users can only access their own resources.
- Avoid exposing direct object references in URLs when possible.
- Use indirect reference mappings or randomized identifiers.
- Implement secure access control validation for every request.

---

# Learning Section

## What I Learned

- User-controlled identifiers can lead to IDOR vulnerabilities.
- Authorization checks must always be enforced server-side.
- Sensitive information should never be accessible by modifying URL parameters.
- IDOR vulnerabilities are common and highly dangerous.
- Even authenticated users should only access resources they are authorized to view.

---

# Tools Used

- Burp Suite
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# Conclusion

This lab demonstrated a Broken Access Control vulnerability caused by insecure handling of user-controlled request parameters.

By modifying the `id` parameter from `wiener` to `carlos`, unauthorized access to another user's account became possible, exposing sensitive information including an API key.
