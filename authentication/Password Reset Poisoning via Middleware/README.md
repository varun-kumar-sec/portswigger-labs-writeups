# Password Reset Poisoning via Middleware

## 📌 Lab Overview

This lab demonstrates a **Password Reset Poisoning** vulnerability caused by improper trust in proxy-related HTTP headers.

The application generated password reset links using the value supplied in the:

```http
X-Forwarded-Host
```

header without validating whether the header originated from a trusted reverse proxy.

As a result, an attacker could manipulate the host used inside password reset emails and cause victims to receive malicious reset links pointing to an attacker-controlled server.

The vulnerability existed because:

- the application trusted user-controlled proxy headers
- password reset URLs were generated using unvalidated host values
- password reset tokens were exposed to attacker-controlled domains
- middleware accepted forwarded headers without verification

This lab focused on:

- password reset poisoning
- middleware trust issues
- X-Forwarded-Host abuse
- account takeover
- password reset workflows
- token theft

---

# 🔍 What is Password Reset Poisoning?

Password reset poisoning occurs when an attacker manipulates the password reset process so that reset tokens are delivered to an attacker-controlled location.

Normally:

```text
Victim Requests Password Reset
↓
Application Generates Reset Link
↓
Email Sent To Victim
↓
Victim Changes Password
```

However, if the host used to generate the reset link can be manipulated:

```text
Victim Requests Password Reset
↓
Attacker Controls Host Header
↓
Reset Link Generated With Attacker Domain
↓
Victim Clicks Link
↓
Token Leaked To Attacker
```

The attacker can then use the stolen token to reset the victim's password.

---

# 🔍 What is the X-Forwarded-Host Header?

`X-Forwarded-Host` is an HTTP header commonly used by reverse proxies and load balancers.

Example:

```http
X-Forwarded-Host: example.com
```

Its purpose is to tell the backend server:

```text
"What hostname did the user originally access?"
```

Example flow:

```text
User
 ↓
Reverse Proxy
 ↓
Backend Application
```

The backend may see:

```http
Host: internal-server.local
X-Forwarded-Host: example.com
```

and generate URLs using:

```text
example.com
```

instead of:

```text
internal-server.local
```

The problem occurs when applications trust this header without verifying that it actually came from a trusted proxy.

An attacker can simply send:

```http
X-Forwarded-Host: attacker.com
```

and trick the application into generating links pointing to:

```text
https://attacker.com
```

instead of the legitimate website.

---

# ⚠ Why Was the Application Vulnerable?

The application generated password reset links using:

```text
Host Information
```

obtained from:

```http
X-Forwarded-Host
```

without validation.

As a result:

```text
Attacker Supplies:
X-Forwarded-Host: attacker.com
```

Application Generates:

```text
https://attacker.com/forgot-password?token=XYZ
```

The reset token becomes exposed to the attacker.

---

# 🎯 Objective

The goal of this lab was to:

- identify the password reset workflow
- test whether proxy headers were trusted
- manipulate password reset emails
- steal Carlos's password reset token
- reset Carlos's password
- authenticate as Carlos

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Poisoning%20via%20Middleware/screenshots/lab11(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Poisoning%20via%20Middleware/screenshots/lab11(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The page contained:

- Username field
- Password field
- Forgot Password option
- Log In button

---

## Screenshot 3 — Initiating Password Reset

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Poisoning%20via%20Middleware/screenshots/lab11(3).png?raw=true)

I clicked:

```text
Forgot Password
```

and was redirected to a page requesting:

```text
Please enter your username or email
```

I entered:

```text
wiener
```

and submitted the request.

---

## Screenshot 4 — Accessing the Email Client

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Poisoning%20via%20Middleware/screenshots/lab11(4).png?raw=true)

The lab provided an:

```text
Exploit Server
```

containing an:

```text
Email Client
```

used for receiving password reset emails.

---

## Screenshot 5 — Viewing the Password Reset Email

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Poisoning%20via%20Middleware/screenshots/lab11(5).png?raw=true)

Inside Wiener's mailbox I received a password reset email containing:

```text
https://0ac5009704ec25f98019031d00290071.web-security-academy.net/forgot-password?temp-forgot-password-token=cg5vqt7lc1x5qqyae2e9ylsnjpbkz3Ol
```

This confirmed that the application generated password reset links using absolute URLs.

---

## Screenshot 6 — Resetting Wiener's Password

![Screenshot 6](screenshot-auth6.png)

After clicking the link, I was redirected to the password reset page.

The page contained:

- New Password
- Confirm New Password

I entered:

```text
password
```

into both fields and submitted the request.

---

## Screenshot 7 — Testing X-Forwarded-Host

![Screenshot 7](screenshot-auth7.png)

I captured the password reset request:

```http
POST /forgot-password
```

Parameter:

```http
username=wiener
```

I then added:

```http
X-Forwarded-Host: www.example.com
```

to determine whether the application trusted the header.

---

## Screenshot 8 — Verifying Host Header Injection

![Screenshot 8](screenshot-auth8.png)

After checking the email again, a new password reset email arrived.

This time the reset link contained:

```text
https://www.example.com/forgot-password?temp-forgot-password-token=4e2ik7nycxf3h9x5vg36bh5ujqjbhdh
```

This confirmed that:

```text
X-Forwarded-Host
```

was being trusted by the application when generating password reset URLs.

---

## Screenshot 9 — Obtaining the Exploit Server Domain

![Screenshot 9](screenshot-auth9.png)

I copied my exploit server URL:

```text
https://exploit-0a0b00f2048725ed8064024b017e0035.exploit-server.net/
```

This server would be used to capture Carlos's reset token.

---

## Screenshot 10 — Poisoning Carlos's Password Reset Request

![Screenshot 10](screenshot-auth10.png)

I modified the captured request:

```http
POST /forgot-password
```

Header:

```http
X-Forwarded-Host: exploit-0a0b00f2048725ed8064024b017e0035.exploit-server.net
```

Parameter:

```http
username=carlos
```

and sent the request.

The application generated a password reset link for Carlos using my exploit server domain.

---

## Screenshot 11 — Capturing Carlos's Reset Token

![Screenshot 11](screenshot-auth11.png)

I opened the exploit server logs and observed a request from the victim:

```text
GET //forgot-password?temp-forgot-password-token=s9uc98u7y7bv529gem3cmluy7q8bje1u
```

This exposed Carlos's password reset token.

The attack chain became:

```text
Poison Reset Email
↓
Victim Clicks Link
↓
Token Sent To Exploit Server
↓
Attacker Steals Token
```

---

## Screenshot 12 — Using Carlos's Token

![Screenshot 12](screenshot-auth12.png)

I first attempted to reuse Wiener's original token:

```text
https://0ac5009704ec25f98019031d00290071.web-security-academy.net/forgot-password?temp-forgot-password-token=cg5vqt7lc1x5qqyae2e9ylsnjpbkz3Ol
```

which resulted in:

```text
Invalid token
```

I then replaced the token value with Carlos's stolen token:

```text
https://0ac5009704ec25f98019031d00290071.web-security-academy.net/forgot-password?temp-forgot-password-token=s9uc98u7y7bv529gem3cmluy7q8bje1u
```

and accessed the page successfully.

---

## Screenshot 13 — Changing Carlos's Password

![Screenshot 13](screenshot-auth13.png)

The application displayed the password reset form.

I entered:

```text
password
```

into both fields and submitted the request.

Carlos's password was successfully changed.

---

## Screenshot 14 — Logging in as Carlos

![Screenshot 14](screenshot-auth14.png)

I returned to the login page and entered:

```text
Username: carlos
Password: password
```

---

## Screenshot 15 — Successful Account Takeover

![Screenshot 15](screenshot-auth15.png)

The login succeeded and I was authenticated as:

```text
carlos
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application trusted:

```http
X-Forwarded-Host
```

without verifying its origin.

This allowed attackers to control:

```text
Password Reset URLs
```

generated by the application.

The complete attack chain was:

```text
Manipulate X-Forwarded-Host
↓
Poison Password Reset Email
↓
Victim Receives Malicious Link
↓
Victim Clicks Link
↓
Token Leaked To Attacker
↓
Password Reset
↓
Account Takeover
```

---

# 💥 Impact

An attacker could potentially:

- steal password reset tokens
- take over user accounts
- bypass authentication
- gain unauthorized access
- compromise administrator accounts
- reset passwords without knowing existing credentials

---

# 🛡 Mitigation

To prevent this issue:

- never trust X-Forwarded-Host directly
- only accept forwarded headers from trusted proxies
- maintain an allowlist of valid domains
- generate reset URLs using server-side configuration
- validate host headers before use
- monitor password reset anomalies
- implement MFA for sensitive accounts

---

# 🧠 Skills Learned

- Authentication Testing
- Password Reset Security
- Password Reset Poisoning
- Middleware Analysis
- X-Forwarded-Host Abuse
- Token Theft
- Account Takeover Techniques
- HTTP Header Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Exploit Server
- Email Client
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how trusting user-controlled proxy headers can completely undermine a password reset workflow.

By identifying that the application trusted the:

```http
X-Forwarded-Host
```

header when generating password reset URLs, I was able to poison Carlos's password reset email, capture his reset token using an attacker-controlled server, and reset his password without knowing the original credentials.

Through this lab, I learned:

- how password reset poisoning attacks work
- why proxy headers should never be blindly trusted
- how X-Forwarded-Host can be abused
- how reset tokens can be stolen through host header manipulation
- how account takeover can occur without exploiting passwords directly

The lab was successfully solved by poisoning the password reset process, capturing Carlos's reset token, resetting his password, and authenticating as Carlos.
