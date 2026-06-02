# Username Enumeration via Response Timing

## 📌 Lab Overview

This lab demonstrates a **Username Enumeration via Response Timing** vulnerability.

Unlike traditional username enumeration, the application returned the same error message for both valid and invalid usernames. However, the backend processed valid usernames differently, resulting in measurable differences in response times.

The vulnerability occurred because:

- valid usernames triggered password verification logic
- invalid usernames were rejected immediately
- response times leaked information about account existence
- attackers could identify valid usernames by measuring server response delays

This lab focused on:

- username enumeration
- timing attacks
- brute-force attacks
- rate-limit bypasses
- authentication weaknesses

---

# 🔍 What is a Timing Attack?

A timing attack occurs when an application unintentionally reveals information through differences in processing time.

For example:

### Invalid Username

```text
1. Username checked
2. User not found
3. Request rejected
```

Response Time:

```text
300ms
```

### Valid Username

```text
1. Username checked
2. User found
3. Password verified
4. Request rejected
```

Response Time:

```text
600ms
```

Even though both requests fail, the response time difference can reveal whether a username exists.

---

# ⚠ Why is This Dangerous?

Attackers can:

- identify valid usernames
- perform targeted brute-force attacks
- bypass generic error messages
- improve password attack success rates

---

# 🎯 Objective

The goal of this lab was to:

- identify a valid username using response timing
- discover the corresponding password
- authenticate successfully
- gain access to the target account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Accessing the Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

I attempted to authenticate using:

```text
Username: abcd
Password: test
```

---

## Screenshot 3 — Generic Authentication Error

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(3).png?raw=true)

After submitting the credentials, the application returned:

```text
Invalid username or password.
```

The error message appeared generic and did not reveal whether the username existed.

---

## Screenshot 4 — Triggering the Rate Limit

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(4).png?raw=true)

I captured the login request:

```http
POST /login
```

Request parameters:

```text
username=abcd
password=test
```

After sending multiple failed login attempts, the application responded with:

```text
You have made too many incorrect login attempts.
Please try again in 30 minute(s)
```

Even valid credentials such as:

```text
wiener:peter
```

were blocked.

This indicated that my IP address had been temporarily rate-limited.

---

## Screenshot 5 — Bypassing the Rate Limit

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(5).png?raw=true)

To bypass the restriction, I added:

```http
X-Forwarded-For: 1
```

to the request.

After sending the request again, the application returned:

```text
Invalid username or password
```

instead of the rate-limit message.

This allowed me to continue testing credentials.

---

# 🔍 What is X-Forwarded-For?

`X-Forwarded-For` is an HTTP header commonly used by proxies and load balancers.

It tells the server the original client IP address.

Example:

```http
X-Forwarded-For: 192.168.1.10
```

Many applications use this header for:

- logging
- analytics
- rate limiting
- access controls

In this lab, the application trusted the header and used it when tracking login attempts.

By changing:

```http
X-Forwarded-For: 1
X-Forwarded-For: 2
X-Forwarded-For: 3
```

I effectively appeared as a different user each time, bypassing the lockout mechanism.

---

## Screenshot 6 — Testing Response Times

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(6).png?raw=true)

To amplify timing differences, I increased the password length:

```text
username=abcd
password=testtesttesttesttesttesttesttesttesttesttesttest
```

When the username was invalid:

```text
Response Time ≈ 361ms
```

The server rejected the request quickly because it did not need to verify the password.

---

## Screenshot 7 — Discovering the Timing Difference

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(7).png?raw=true)

I then tested a valid username.

The response time increased significantly:

```text
Response Time ≈ 626ms
```

This happened because:

```text
Username Exists
↓
Password Verification Starts
↓
Additional Processing Time
```

The longer response revealed a valid username.

---

## Screenshot 8 — Configuring Burp Intruder

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(8).png?raw=true)

I sent the request to Burp Intruder and selected two payload positions:

```http
X-Forwarded-For: §3§

username=§abcd§
password=testtesttesttesttesttesttesttesttesttesttesttest
```

### Payload Set 1

```text
Payload Type: Numbers
Range: 3 → 203
```

Used to generate unique IP addresses.

### Payload Set 2

```text
Usernames provided by the lab
```

---

### What is a Pitchfork Attack?

A Pitchfork attack uses multiple payload sets simultaneously.

Example:

```text
Payload Set 1: 1 2 3
Payload Set 2: alice bob carlos
```

Requests generated:

```text
1 + alice
2 + bob
3 + carlos
```

Each payload advances together.

This is different from:

### Sniper

```text
Tests one position at a time
```

### Cluster Bomb

```text
Tests every possible combination
```

Pitchfork was useful here because every username required a unique IP address.

---

## Screenshot 9 — Identifying the Valid Username

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(9).png?raw=true)

After launching the attack, I sorted the results by:

```text
Response Received
```

The request with the highest response time corresponded to:

```text
Username: auth
```

This indicated that:

```text
auth
```

was a valid username.

---

## Screenshot 10 — Brute-Forcing the Password

![Screenshot 10](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(10).png?raw=true)

After identifying the username, I modified the attack:

```http
username=auth
password=§test§
```

I also updated the IP range:

```text
203 → 303
```

to avoid reusing previously rate-limited IPs.

The password list provided by the lab was loaded into Intruder.

A Pitchfork attack was launched again.

---

## Screenshot 11 — Discovering the Password

![Screenshot 11](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(11).png?raw=true)

After reviewing the results, I identified the correct password:

```text
Username: auth
Password: montana
```

---

## Screenshot 12 — Successful Authentication

![Screenshot 12](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(12).png?raw=true)

![Screenshot 13](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Response%20Timing/screenshots/lab5(13).png?raw=true)

Using Burp Suite's **Request in Browser** feature, I replayed the successful authentication request.

The login succeeded and I was authenticated as:

```text
auth
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability occurred because the application processed:

```text
Valid Username
```

and

```text
Invalid Username
```

differently.

Valid usernames triggered password verification, increasing response times.

This timing difference leaked information about account existence.

---

# 💥 Impact

An attacker can potentially:

- identify valid usernames
- bypass generic error messages
- conduct targeted password attacks
- improve brute-force efficiency
- perform account takeover attacks

---

# 🛡 Mitigation

To prevent timing-based username enumeration:

- use constant-time authentication routines
- normalize authentication response times
- return identical responses for all failures
- implement robust rate limiting
- avoid trusting X-Forwarded-For headers directly
- deploy MFA where possible

---

# 🧠 Skills Learned

- Username Enumeration
- Timing Attacks
- Authentication Testing
- Burp Intruder
- Pitchfork Attacks
- Rate Limit Bypass
- X-Forwarded-For Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how response timing differences can reveal valid usernames even when authentication messages appear identical.

By measuring server response times, manipulating the `X-Forwarded-For` header to bypass rate limiting, and using Burp Intruder with a Pitchfork attack, I successfully identified a valid username and its password.

Through this lab, I learned:

- how timing attacks work
- why response consistency is important
- how rate limiting can be bypassed
- how Burp Intruder can automate timing-based attacks

The lab was successfully solved by exploiting response timing differences to enumerate usernames and discover valid credentials.
