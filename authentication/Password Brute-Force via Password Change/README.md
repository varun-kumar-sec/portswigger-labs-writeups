# Password Brute-Force via Password Change

## 📌 Lab Overview

This lab demonstrates a **Password Brute-Force via Password Change** vulnerability.

The application contained a flaw in its password change functionality that leaked information about whether the supplied **current password** was correct.

Instead of returning the same generic error for all failures, the application returned different responses depending on which validation step failed.

This allowed an attacker to:

- identify valid passwords
- brute force another user's current password
- bypass account lockout protections
- gain unauthorized access to another account

This lab focused on:

- authentication weaknesses
- password change logic flaws
- response-based brute forcing
- error message analysis
- Burp Intruder automation

---

# 🔍 What is Password Brute Forcing?

Password brute forcing is the process of systematically trying multiple passwords until the correct one is discovered.

Example:

```text
Password Attempt 1 → Incorrect
Password Attempt 2 → Incorrect
Password Attempt 3 → Correct
```

Applications should ensure that attackers cannot distinguish:

```text
Wrong Password
vs
Correct Password but Different Error
```

because different responses can leak information useful for brute-force attacks.

---

# ⚠ Why Was the Password Change Function Vulnerable?

The password change feature validated requests in multiple stages:

```text
1. Verify Current Password
2. Verify New Passwords Match
3. Change Password
```

Because different validation failures returned different messages, an attacker could determine whether the current password was correct.

This created an oracle that allowed password brute forcing.

---

# 🎯 Objective

The goal of this lab was to:

- analyze the password change functionality
- identify information leakage through error messages
- brute force Carlos's current password
- authenticate as Carlos
- gain access to the account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Brute-Force%20via%20Password%20Change/screenshots/lab13(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Logging in as Wiener

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Brute-Force%20via%20Password%20Change/screenshots/lab13(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and clicked **Log in**.

---

## Screenshot 3 — Accessing Password Change Functionality

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Brute-Force%20via%20Password%20Change/screenshots/lab13(3).png?raw=true)

After successful authentication, I was logged in as:

```text
wiener
```

The page contained the following fields:

```text
Email
Current Password
New Password
Confirm New Password
```

I tested the functionality using:

```text
Current Password: peter
New Password: 123
Confirm New Password: 123
```

---

## Screenshot 4 — Verifying Password Change

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Brute-Force%20via%20Password%20Change/screenshots/lab13(4).png?raw=true)

After changing the password, I returned to the login page and successfully logged in using:

```text
Username: wiener
Password: 123
```

This confirmed that the password change feature was functioning correctly.

---

## Screenshot 5 — Observing Login Lockout

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Brute-Force%20via%20Password%20Change/screenshots/lab13(5).png?raw=true)

I intentionally entered incorrect credentials multiple times.

After repeated failures, the application responded with:

```text
You have made too many incorrect login attempts.
Please try again after 1 minute.
```

This indicated that brute-force protection existed on the login page.

---

## Screenshot 6 — Discovering the Information Leak

![Screenshot 6](screenshot-auth6.png)

After waiting for the lockout period to expire, I logged back in as Wiener.

I then intentionally submitted:

```text
Current Password: 123
New Password: 123
Confirm New Password: 1233
```

The application returned:

```text
New password do not match
```

This response revealed something important.

### Three Possible Scenarios

#### Scenario 1

```text
Correct Current Password
Matching New Passwords
```

Result:

```text
Password Changed Successfully
```

---

#### Scenario 2

```text
Incorrect Current Password
Matching New Passwords
```

Result:

```text
Too Many Incorrect Login Attempts
```

or

```text
Current Password Incorrect
```

depending on the application's logic.

---

#### Scenario 3

```text
Correct Current Password
Mismatched New Passwords
```

Result:

```text
New password do not match
```

This was the critical discovery.

Because the application checked the current password before validating the new passwords, receiving:

```text
New password do not match
```

proved that:

```text
Current Password = Correct
```

This behavior could be abused to brute force another user's password.

---

# 🔍 Why This Error Message Leaks Password Validity

Consider the validation flow:

```text
Step 1 → Verify Current Password
Step 2 → Compare New Password Fields
Step 3 → Change Password
```

If:

```text
Current Password = Wrong
```

the request fails immediately.

However, if:

```text
Current Password = Correct
```

the application proceeds to Step 2.

Therefore:

```text
New password do not match
```

can only occur when:

```text
Current Password = Valid
```

This turns the password change feature into a password verification oracle.

---

## Screenshot 7 — Configuring Burp Intruder

![Screenshot 7](screenshot-auth7.png)

I captured the password change request:

```http
POST /my-account/change-password
```

Parameters:

```text
username=carlos
current-password=test
new-password-1=123
new-password-2=1233
```

I sent the request to Burp Intruder and selected:

```text
current-password
```

as the payload position.

Modified request:

```text
username=carlos
current-password=§test§
new-password-1=123
new-password-2=1233
```

I configured:

### Grep Match

```text
New password do not match
```

### Payload List

All passwords supplied by the lab.

I then launched the attack.

---

## Screenshot 8 — Identifying Carlos's Password

![Screenshot 8](screenshot-auth8.png)

After Intruder completed the attack, I reviewed the results.

One request contained:

```text
New password do not match
```

in the response.

This indicated that the supplied current password was valid.

The discovered password was:

```text
amanda
```

---

## Screenshot 9 — Logging in as Carlos

![Screenshot 9](screenshot-auth9.png)

I returned to the login page and entered:

```text
Username: carlos
Password: amanda
```

---

## Screenshot 10 — Successful Authentication

![Screenshot 10](screenshot-auth10.png)

The login succeeded and I was authenticated as:

```text
carlos
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application returned different responses for different validation failures.

Example:

```text
Wrong Current Password
↓
Error A
```

```text
Correct Current Password
+
Mismatched New Passwords
↓
Error B
```

These differences allowed attackers to determine whether a password guess was correct.

Authentication-related functions should never expose this information.

---

# 💥 Impact

An attacker could potentially:

- brute force user passwords
- verify password guesses
- bypass login rate limits
- gain unauthorized account access
- compromise sensitive user data

---

# 🛡 Mitigation

To prevent this issue:

- return identical error messages for all failures
- avoid revealing validation order
- enforce rate limiting on password change endpoints
- require re-authentication before password changes
- implement MFA
- monitor password reset and password change abuse

Example:

```text
Invalid credentials
```

should be returned regardless of which validation failed.

---

# 🧠 Skills Learned

- Authentication Testing
- Password Change Workflow Analysis
- Response-Based Enumeration
- Burp Intruder
- Error Message Analysis
- Password Verification Oracles
- Brute Force Techniques

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how seemingly harmless error messages can leak sensitive authentication information.

By analyzing the password change workflow, identifying differences in server responses, and using Burp Intruder to automate password testing, I was able to discover Carlos's password without attacking the login page directly.

Through this lab, I learned:

- how password verification oracles work
- why consistent error handling is important
- how authentication logic flaws enable brute-force attacks
- how Burp Intruder can automate password discovery

The lab was successfully solved by exploiting information leakage in the password change functionality and authenticating as Carlos.
