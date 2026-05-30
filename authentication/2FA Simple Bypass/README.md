# 2FA Simple Bypass

## 📌 Lab Overview

This lab demonstrates a **Two-Factor Authentication (2FA) Bypass** vulnerability.

The vulnerability occurred because:
- the application failed to properly enforce the second authentication step
- sensitive pages could be accessed directly through predictable URLs
- the application trusted client-side navigation instead of validating authentication state

This lab focused on:
- 2FA bypass
- authentication flaws
- access control weaknesses
- forced browsing
- authentication workflow analysis

---

# 🔍 What is 2FA?

**2FA (Two-Factor Authentication)** is an additional security layer used after entering a username and password.

Normally authentication happens in two steps:

### Step 1 — Something You Know

```text
Username + Password
```

### Step 2 — Something You Have

```text
Email Code
SMS Code
Authenticator App
Hardware Token
```

Example:

```text
1. Enter username and password
2. Receive verification code
3. Enter verification code
4. Access account
```

Even if an attacker steals a password, they should not be able to access the account without the second factor.

---

# ⚠ Why 2FA Bypass is Dangerous

If 2FA can be bypassed:

- attackers can access accounts without verification codes
- account takeover becomes easier
- the second security layer becomes useless
- sensitive user data can be exposed

---

# 🎯 Objective

The goal of this lab was to:
- understand the 2FA workflow
- identify weaknesses in the authentication process
- bypass the second authentication factor
- gain access to another user's account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(1).png?raw=true)

The application initially displayed:
- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Logging In

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided the following credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and proceeded with authentication.

---

## Screenshot 3 — 2FA Verification Page

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(3).png?raw=true)

After successful login, I was redirected to the second authentication step.

The page displayed:

```text
Please enter your 4-digit security code
```

A login button was also available.

At the top of the page there was an **Email Client** option provided by the lab.

---

## Screenshot 4 — Retrieving the Security Code

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(4).png?raw=true)

I clicked the **Email Client** button and obtained the verification code:

```text
0979
```

---

## Screenshot 5 — Completing Verification

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(5).png?raw=true)

I entered the verification code:

```text
0979
```

and clicked the login button.

---

## Screenshot 6 — Successful Login as Wiener

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(6).png?raw=true)

After verification, I was successfully logged in as:

```text
wiener
```

This confirmed the normal authentication workflow.

---

## Screenshot 7 — Analyzing the 2FA Request

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(7).png?raw=true)

I captured the 2FA verification request:

```http
POST /login2
```

While inspecting the response, I found:

```http
Location: /my-account?id=wiener
```

---

# 🔍 Why This Was Interesting

The application redirected users directly to:

```http
/my-account?id=wiener
```

after successful 2FA verification.

This suggested that user account pages might be accessible directly through the URL.

The application appeared to rely on navigation logic instead of properly validating whether the second authentication step had been completed.

---

## Screenshot 8 — Directly Accessing Another User's Account

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Simple%20Bypass/screenshots/lab2(8).png?raw=true)

I copied the URL from the response and modified it:

```text
https://LAB-ID.web-security-academy.net/my-account?id=carlos
```

Instead of:

```text
/my-account?id=wiener
```

I requested:

```text
/my-account?id=carlos
```

directly through the browser.

---

## Screenshot 9 — Successful 2FA Bypass

![Screenshot 9](screenshot-auth9.png)

The application granted access to:

```text
carlos
```

without requiring Carlos's verification code.

As a result:
- the second authentication factor was bypassed
- another user's account became accessible
- the lab was successfully solved

---

# ⚠ Why This Vulnerability Exists

The vulnerability occurred because the application trusted URL parameters and authentication state incorrectly.

Instead of verifying:

```text
Has the current user completed 2FA?
```

the application simply served:

```text
/my-account?id=<user>
```

when requested.

This allowed attackers to skip part of the authentication process.

---

# 💥 Impact

An attacker can potentially:

- bypass multi-factor authentication
- access unauthorized accounts
- perform account takeover
- view sensitive information
- modify user data

---

# 🛡 Mitigation

To prevent 2FA bypass vulnerabilities:

- enforce server-side verification of completed 2FA
- validate authentication state on every request
- never trust URL parameters for authorization decisions
- bind authenticated sessions to verified users
- perform proper access control checks

Example:

```text
User requests /my-account?id=carlos
```

The server should verify:

```text
Is the authenticated user actually Carlos?
Has Carlos completed 2FA?
```

before granting access.

---

# 🧠 Skills Learned

- Two-Factor Authentication (2FA)
- Authentication Workflow Analysis
- Access Control Testing
- Forced Browsing
- Authentication Bypass
- Session Validation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improper enforcement of Two-Factor Authentication can lead to authentication bypass.

By analyzing the 2FA verification process and observing the redirect location, I discovered that account pages could be accessed directly through predictable URLs.

By modifying:

```text
/ my-account?id=wiener
```

to:

```text
/ my-account?id=carlos
```

I was able to bypass the intended authentication workflow and access another user's account.

Through this lab, I learned:
- how 2FA works
- how authentication workflows can fail
- why server-side verification is critical
- how authentication bypass vulnerabilities occur

The lab was successfully solved by bypassing the second authentication factor and directly accessing another user's account.
