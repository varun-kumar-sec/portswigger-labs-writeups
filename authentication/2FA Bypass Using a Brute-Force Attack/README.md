# 2FA Bypass Using a Brute-Force Attack

## 📌 Lab Overview

This lab demonstrates a **2FA Bypass via Brute Force** vulnerability.

The application implemented a **Two-Factor Authentication (2FA)** mechanism that required users to enter a **4-digit security code** after successfully providing valid credentials.

Although the username and password authentication was correct, the second authentication factor was weak because:

- the security code consisted of only 4 digits
- no effective rate limiting existed
- no account lockout was implemented
- valid authenticated sessions could be reused during brute forcing

As a result, an attacker who already possessed valid credentials could brute force the second authentication factor and gain full account access.

This lab focused on:

- Two-Factor Authentication (2FA)
- Authentication Testing
- Session Handling Rules
- Burp Suite Macros
- Session Management
- Brute Force Attacks
- Security Code Enumeration

---

# 🔍 What is Two-Factor Authentication (2FA)?

Two-Factor Authentication (2FA) is an additional security layer that requires users to provide a second form of verification after entering their username and password.

Typical authentication flow:

```text
Username + Password
        ↓
Security Code / OTP
        ↓
Access Granted
```

Even if an attacker knows the correct password, they should not be able to access the account without the second factor.

Common second factors include:

- OTP applications
- SMS verification codes
- Email verification codes
- Hardware security keys
- Authenticator applications

---

# ⚠ Why Was the 2FA Protection Vulnerable?

The application used a:

```text
4-digit security code
```

which provides only:

```text
0000 → 9999
```

or:

```text
10,000 possible combinations
```

Additionally:

- there was no lockout mechanism
- there was no rate limiting
- brute force attempts were not restricted
- authenticated sessions could be reused

This made it possible to systematically guess every possible code until the correct one was found.

---

# 🎯 Objective

The goal of this lab was to:

- authenticate using valid credentials
- reach the 2FA verification page
- automate session handling
- brute force the 4-digit security code
- gain access to Carlos's account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Bypass%20Using%20a%20Brute-Force%20Attack/screenshots/lab14(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Login as Carlos

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Bypass%20Using%20a%20Brute-Force%20Attack/screenshots/lab14(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: carlos
Password: montoya
```

I entered the credentials and clicked:

```text
Log In
```

---

## Screenshot 3 — 2FA Verification Page

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/2FA%20Bypass%20Using%20a%20Brute-Force%20Attack/screenshots/lab14(3).png?raw=true)

After successful authentication, I was redirected to the second authentication step.

The page requested:

```text
Please enter your 4-digit security code
```

This confirmed:

```text
Username = Correct
Password = Correct
```

but access was still protected by the second factor.

---

# 🔍 What are Session Handling Rules?

Burp Suite's **Session Handling Rules** allow Burp to automatically perform actions before sending requests.

Examples include:

- logging in automatically
- refreshing sessions
- obtaining CSRF tokens
- updating cookies
- executing macros

Instead of manually repeating login requests thousands of times, Burp can automate the entire process.

In this lab:

```text
Brute Force Request
        ↓
Automatically Execute Login Macro
        ↓
Get Fresh Session
        ↓
Submit Security Code
```

This ensured every request used a valid authenticated session.

---

## Screenshot 4 — Session Handling Rules Configuration

![Screenshot 4](screenshot-auth4.png)

I opened:

```text
Project Options
→ Sessions
→ Session Handling Rules
```

A custom rule named:

```text
Rule 1
```

was configured.

This rule would automatically execute a predefined macro before each brute-force request.

---

## Screenshot 5 — Session Handling Rule Editor

![Screenshot 5](screenshot-auth5.png)

Inside the:

```text
Session Handling Rule Editor
```

I configured the rule action:

```text
Run Macro: Macro1
```

---

# 🔍 What is a Session Handling Rule Editor?

The Session Handling Rule Editor defines:

```text
When should Burp run the rule?
What actions should Burp perform?
```

Examples:

```text
Run a Macro
Update Cookies
Update CSRF Tokens
Modify Parameters
Refresh Sessions
```

In this lab:

```text
Rule Triggered
        ↓
Run Macro1
        ↓
Get Fresh Session
        ↓
Continue Attack
```

---

# 🔍 What is a Macro?

A Macro is a predefined sequence of HTTP requests that Burp can execute automatically.

Example:

```text
GET /login
POST /login
GET /account
```

Instead of manually performing these actions repeatedly:

```text
Burp executes them automatically
```

Macros are commonly used for:

- logging in automatically
- fetching CSRF tokens
- maintaining sessions
- bypassing workflow restrictions

---

## Screenshot 6 — Session Handling Action Editor

![Screenshot 6](screenshot-auth6.png)

Inside:

```text
Session Handling Action Editor
```

I selected:

```text
Macro1
```

as the macro that Burp should execute.

Additionally:

```text
Update Current Request
With Parameters From Macro
```

and:

```text
Update Cookies From Cookie Jar
```

were enabled.

This allowed Burp to automatically use fresh authentication data.

---

## Screenshot 7 — Macro Editor Configuration

![Screenshot 7](screenshot-auth7.png)

Inside the Macro Editor, I added three requests:

```text
GET /login
POST /login
GET /login2
```

---

### Why Were These Requests Needed?

#### Request 1

```text
GET /login
```

Purpose:

```text
Retrieve a fresh CSRF token
```

---

#### Request 2

```text
POST /login
```

Purpose:

```text
Authenticate as Carlos
```

using:

```text
carlos : montoya
```

---

#### Request 3

```text
GET /login2
```

Purpose:

```text
Reach the 2FA page
```

and obtain a valid session for the security code verification.

The complete workflow became:

```text
GET /login
        ↓
POST /login
        ↓
GET /login2
        ↓
Ready to Submit Security Code
```

---

## Screenshot 8 — Testing the Macro

![Screenshot 8](screenshot-auth8.png)

I clicked:

```text
Test Macro
```

Burp successfully executed all requests.

A fresh:

```text
CSRF Token
```

was retrieved automatically.

This confirmed that the macro was working correctly.

---

## Screenshot 9 — Brute Forcing the Security Code

![Screenshot 9](screenshot-auth9.png)

After configuring session handling, I launched the brute-force attack against the security code field.

Burp Intruder tested:

```text
0000
0001
0002
...
9999
```

During the attack, Burp automatically:

```text
Run Macro
↓
Obtain Session
↓
Submit Security Code
```

for every request.

Eventually, one response returned:

```http
302 Found
```

which indicated successful authentication.

The valid security code was:

```text
0187
```

---

# 🔍 Why Was the 302 Response Important?

Normally:

```text
Wrong Security Code
↓
Stay on Verification Page
↓
200 OK
```

However:

```text
Correct Security Code
↓
Authentication Successful
↓
302 Redirect
```

The redirect revealed the correct 2FA code.

---

## Screenshot 10 — Replaying the Successful Response

![Screenshot 10](screenshot-auth10.png)

Using Burp Suite's feature:

```text
Show Response In Browser
```

I copied the authenticated response URL.

---

## Screenshot 11 — Successful Login as Carlos

![Screenshot 11](screenshot-auth11.png)

After pasting the URL into the browser:

```text
Carlos Account
```

was successfully accessed.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- the security code space was too small
- no rate limiting existed
- no account lockout existed
- brute force attempts were unrestricted
- sessions could be reused automatically

As a result:

```text
Attacker Knows Password
        ↓
Brute Force 2FA Code
        ↓
Gain Full Access
```

---

# 💥 Impact

An attacker could potentially:

- bypass Two-Factor Authentication
- gain unauthorized access to accounts
- compromise sensitive information
- perform account takeover attacks
- defeat the purpose of 2FA protection

---

# 🛡 Mitigation

To prevent this vulnerability:

- implement rate limiting
- lock accounts after repeated failures
- use longer verification codes
- expire codes quickly
- enforce MFA throttling
- monitor brute-force activity
- require CAPTCHA after repeated attempts

---

# 🧠 Skills Learned

- Two-Factor Authentication Testing
- Session Handling Rules
- Burp Suite Macros
- Session Management
- Authentication Workflow Analysis
- Brute Force Attacks
- CSRF Token Handling
- Intruder Automation

---

# 🧰 Tools Used

- Burp Suite Professional
- Burp Intruder
- Burp Repeater
- Burp Session Handling Rules
- Burp Macros
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how weakly implemented Two-Factor Authentication can be bypassed through brute force attacks.

By configuring Burp Suite Session Handling Rules and Macros, I automated the login workflow, maintained valid sessions, and ensured fresh CSRF tokens were used during every request.

Once the workflow was automated, I brute forced the 4-digit security code until the application returned a successful authentication response.

Through this lab, I learned:

- how 2FA workflows operate
- how Burp Session Handling Rules work
- how Burp Macros automate authentication flows
- how CSRF tokens can be refreshed automatically
- how brute force attacks can defeat weak 2FA implementations

The lab was successfully solved by brute forcing the 2FA security code and gaining access to Carlos's account.
