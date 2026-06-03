# Broken Brute Force Protection, IP Block

## 📌 Lab Overview

This lab demonstrates a **Broken Brute Force Protection** vulnerability.

The application attempted to protect user accounts by temporarily locking them after multiple failed login attempts. However, the protection mechanism contained a logical flaw that allowed an attacker to bypass the lockout and continue brute-forcing passwords.

The vulnerability existed because:

- the account was locked after two failed login attempts
- a successful login reset the failed-attempt counter
- an attacker-controlled valid account could be used to continuously reset the lockout state
- password brute forcing remained possible despite the protection mechanism

This lab focused on:

- authentication weaknesses
- brute force attacks
- lockout bypass techniques
- Burp Intruder automation
- Pitchfork attacks

---

# 🔍 What is Brute Force Protection?

Brute force protection is a security mechanism designed to prevent attackers from repeatedly guessing passwords.

Common protections include:

- account lockouts
- IP blocking
- rate limiting
- CAPTCHA challenges
- MFA enforcement

Example:

```text
Attempt 1 → Incorrect Password
Attempt 2 → Incorrect Password
Attempt 3 → Account Locked
```

The goal is to slow down or completely stop password guessing attacks.

---

# ⚠ Why Was the Protection Broken?

The application locked an account after two failed login attempts.

However:

```text
Failed Attempt
Failed Attempt
Successful Login
```

reset the counter.

This allowed an attacker to continuously alternate between:

```text
Carlos → Wrong Password
Carlos → Wrong Password
Wiener → Correct Password
```

which prevented the lockout from ever becoming permanent.

---

# 🎯 Objective

The goal of this lab was to:

- bypass the login protection mechanism
- brute force Carlos's password
- authenticate as Carlos
- gain access to the account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Broken%20Brute%20Force%20Protection,%20IP%20Block/screenshots/lab6(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Logging in as Wiener

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Broken%20Brute%20Force%20Protection,%20IP%20Block/screenshots/lab6(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: wiener
Password: peter
```

I entered these credentials and attempted to log in.

---

## Screenshot 3 — Successful Login

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Broken%20Brute%20Force%20Protection,%20IP%20Block/screenshots/lab6(3).png?raw=true)

The login was successful and I was authenticated as:

```text
wiener
```

This account would later be used to bypass the brute-force protection.

---

## Screenshot 4 — Testing Carlos Login

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Broken%20Brute%20Force%20Protection,%20IP%20Block/screenshots/lab6(4).png?raw=true)

After logging out, I returned to the login page.

The target account was:

```text
carlos
```

I attempted to log in using:

```text
Username: carlos
Password: 1234
```

---

## Screenshot 5 — Incorrect Password Message

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Broken%20Brute%20Force%20Protection,%20IP%20Block/screenshots/lab6(5).png?raw=true)

The application responded with:

```text
Incorrect password
```

indicating that the username existed but the password was wrong.

---

## Screenshot 6 — Discovering the Lockout Logic

![Screenshot 6](screenshot-auth6.png)

I captured the login request:

```http
POST /login
```

Parameters:

```text
username=carlos
password=1234
```

After sending the request twice through Repeater:

```text
Incorrect password
```

was returned.

However, on the third attempt the application responded with:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute(s).
```

This revealed the lockout threshold.

The logic appeared to be:

```text
Attempt 1 → Allowed
Attempt 2 → Allowed
Attempt 3 → Locked
```

---

## Screenshot 7 — Verifying Lockout Reset

![Screenshot 7](screenshot-auth7.png)

After waiting one minute, I modified the request and used valid credentials:

```text
username=wiener
password=peter
```

The response returned:

```http
302 Found
```

which confirmed:

```text
Successful Login
↓
Failed Attempt Counter Reset
```

This behavior became the key to bypassing the protection mechanism.

---

## Screenshot 8 — Preparing the Username List

![Screenshot 8](screenshot-auth8.png)

After understanding the logic, I modified my username list.

The pattern used was:

```text
carlos
carlos
wiener
carlos
carlos
wiener
...
```

The idea was:

```text
2 Incorrect Attempts
↓
1 Successful Login (wiener:peter)
↓
Counter Reset
```

This prevented Carlos's account from remaining locked.

---

## Screenshot 9 — Preparing the Password List

![Screenshot 9](screenshot-auth9.png)

I also modified the password list.

Example pattern:

```text
123456
password
peter
qwerty
12345678
peter
...
```

Here:

```text
peter
```

corresponded to the Wiener account.

This ensured every third request was a successful login.

---

## Screenshot 10 — Configuring Burp Intruder

![Screenshot 10](screenshot-auth10.png)

I sent the login request to Burp Intruder.

Selected payload positions:

```http
username=§admin§
password=§test§
```

Attack Type:

```text
Pitchfork
```

The modified username and password lists were loaded into their respective payload positions.

---

# 🔍 What is a Pitchfork Attack?

Pitchfork attacks use multiple payload sets simultaneously.

Example:

```text
Payload Set 1:
carlos
carlos
wiener

Payload Set 2:
123456
password
peter
```

Generated requests:

```text
carlos : 123456
carlos : password
wiener : peter
```

Each payload advances together.

This makes Pitchfork ideal when multiple parameters must remain synchronized.

---

## Screenshot 11 — Resource Pool Configuration

![Screenshot 11](screenshot-auth11.png)

Before launching the attack, I created a custom resource pool.

Configuration:

```text
Maximum Concurrent Requests = 1
```

### Why Was This Important?

If Intruder sends multiple requests simultaneously:

```text
Request 1
Request 2
Request 3
```

the account may become locked before the successful Wiener login can reset the counter.

By limiting Intruder to:

```text
1 request at a time
```

the sequence remains:

```text
Carlos Wrong
Carlos Wrong
Wiener Correct
Carlos Wrong
Carlos Wrong
Wiener Correct
```

ensuring the lockout is continuously bypassed.

In short:

```text
Resource Pool = Controls Attack Speed and Concurrency
```

Setting concurrency to **1** guarantees the requests are processed in the intended order.

---

## Screenshot 12 — Discovering Carlos's Password

![Screenshot 12](screenshot-auth12.png)

After launching the attack, I reviewed the results.

The correct credentials were identified as:

```text
Username: carlos
Password: 123qwe
```

---

## Screenshot 13 — Replaying the Successful Request

![Screenshot 13](screenshot-auth13.png)

Using Burp Suite's:

```text
Request in Browser
```

feature, I copied the successful authenticated request and pasted it into the browser.

---

## Screenshot 14 — Successful Login as Carlos

![Screenshot 14](screenshot-auth14.png)

The login succeeded and I was authenticated as:

```text
carlos
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

```text
Failed Login Attempts
```

were reset immediately after a successful login.

An attacker with access to any valid account could continuously reset the counter and continue brute forcing another account.

The lockout mechanism therefore failed to provide effective protection.

---

# 💥 Impact

An attacker could potentially:

- bypass account lockouts
- brute force passwords
- perform credential stuffing attacks
- gain unauthorized account access
- compromise user accounts

---

# 🛡 Mitigation

To prevent this issue:

- implement account lockouts independent of successful logins
- apply lockouts per target account
- use rate limiting
- enforce MFA
- monitor authentication anomalies
- require CAPTCHA after repeated failures
- track brute-force attempts across sessions and IPs

---

# 🧠 Skills Learned

- Authentication Testing
- Brute Force Attacks
- Burp Intruder
- Pitchfork Attacks
- Lockout Bypass Techniques
- Resource Pool Configuration
- Login Workflow Analysis

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how poorly implemented brute force protection can be bypassed through logical flaws in the authentication workflow.

By analyzing the lockout behavior, identifying that successful logins reset the failed-attempt counter, and synchronizing login attempts using a Pitchfork attack, I was able to continuously bypass the account lockout mechanism.

Through this lab, I learned:

- how brute force protections work
- how lockout mechanisms can fail
- how to synchronize payloads using Pitchfork attacks
- how Burp Intruder resource pools affect attack reliability

The lab was successfully solved by exploiting a flawed lockout implementation and brute-forcing Carlos's password.
