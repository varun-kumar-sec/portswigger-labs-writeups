# Brute-Forcing a Stay-Logged-In Cookie

## 📌 Lab Overview

This lab demonstrates a **weak persistent login implementation** where the application uses a predictable "stay logged in" cookie.

Instead of securely storing a random token on the server side, the application generated the cookie using:

```text
base64(username:md5(password))
```

Because the cookie was derived directly from the user's credentials, an attacker could:

- reverse engineer the cookie structure
- crack the password hash
- generate valid cookies for other users
- gain unauthorized access without knowing the victim's password

This lab focused on:

- authentication weaknesses
- insecure persistent login mechanisms
- cookie analysis
- MD5 hash cracking
- Burp Intruder payload processing
- account takeover

---

# 🔍 What is a Stay-Logged-In Cookie?

Many websites offer a:

```text
☑ Stay Logged In
```

option that allows users to remain authenticated even after closing their browser.

A secure implementation typically works like this:

```text
User Logs In
      │
      ▼
Server Generates Random Token
      │
      ▼
Token Stored Server-Side
      │
      ▼
Token Sent as Cookie
```

Example:

```text
stay-logged-in=RANDOM_LONG_TOKEN
```

The server then validates the token against its database.

---

## Secure Example

```text
stay-logged-in=7af92cbd7d82caa90f85f4e9c1
```

The token contains:

- no username
- no password
- no predictable data

---

## Insecure Example

```text
base64(username:md5(password))
```

Example:

```text
base64(wiener:md5(peter))
```

Because the cookie contains information derived from user credentials, attackers can reverse engineer the format and generate their own valid cookies.

This is exactly what happened in this lab.

---

# ⚠ Why Does This Vulnerability Exist?

The vulnerability exists because the application uses:

```text
username + password hash
```

as the authentication token.

The application trusts the cookie itself instead of verifying a random server-side session token.

As a result:

```text
If attacker knows cookie format
            │
            ▼
Can generate cookies for other users
            │
            ▼
Authentication bypass
```

The problem becomes even worse because:

```text
MD5
```

is a weak hashing algorithm that can often be cracked using public rainbow tables.

---

# 🎯 Objective

The goal of this lab was to:

- analyze the stay-logged-in cookie
- identify how it was generated
- crack the stored password hash
- create a valid cookie for Carlos
- gain access to Carlos's account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(1).png?raw=true)

The application initially displayed a normal webpage containing:

- My Account button

---

## Screenshot 2 — Logging In with Stay Logged In Enabled

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(2).png?raw=true)

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

before submitting the login request.

---

## Screenshot 3 — Successful Login

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(3).png?raw=true)

The login succeeded and I was authenticated as:

```text
wiener
```

The account page contained an:

```text
Update Email
```

functionality.

---

## Screenshot 4 — Analyzing the Cookie

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(4).png?raw=true)

I captured the authenticated request:

```http
GET /my-account?id=wiener
```

The request contained:

```http
Cookie:
stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3NGQ0M2E2MDExZTllYmJhNmNhNzcw
```

Using Burp's Inspector panel, I decoded the cookie from Base64 and obtained:

```text
wiener:51dc30ddc474d43a6011e9ebba6ca770
```

This immediately revealed two components:

```text
wiener
```

and

```text
51dc30ddc474d43a6011e9ebba6ca770
```

The second value looked like an MD5 hash.

---

# 🔍 Understanding the Cookie Structure

The cookie followed the format:

```text
base64(username:md5(password))
```

For Wiener:

```text
Username = wiener
Password = peter
```

The application generated:

```text
md5(peter)
```

↓

```text
51dc30ddc474d43a6011e9ebba6ca770
```

↓

```text
wiener:51dc30ddc474d43a6011e9ebba6ca770
```

↓

```text
Base64 Encode
```

↓

```text
stay-logged-in=d2llbmVyOjUxZGMzMGRkYzQ3NGQ0M2E2MDExZTllYmJhNmNhNzcw
```

The authentication process can therefore be visualized as:

```text
Password
   │
   ▼
MD5 Hash
   │
   ▼
username:hash
   │
   ▼
Base64 Encode
   │
   ▼
Cookie
```

Once this format was discovered, the only unknown value became:

```text
Carlos's Password
```

---

## Screenshot 5 — Cracking the MD5 Hash

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(5).png?raw=true)

I submitted the MD5 hash to:

```text
CrackStation
```

The hash was successfully cracked and revealed:

```text
peter
```

This confirmed my theory that the cookie was generated using:

```text
base64(username:md5(password))
```

At this point I knew that if I could discover Carlos's password, I could generate a valid stay-logged-in cookie for him.

---

# 🔍 Why Was MD5 a Problem?

MD5 is considered cryptographically broken because:

- it is extremely fast
- rainbow tables exist
- billions of hashes have already been cracked

Example:

```text
md5(password)
```

↓

```text
5f4dcc3b5aa765d61d8327deb882cf99
```

Attackers can simply search public databases to recover the original password.

Modern applications should use:

- bcrypt
- Argon2
- PBKDF2

instead of MD5.

---

## Screenshot 6 — Sending the Request to Intruder

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(6).png?raw=true)

After understanding the cookie structure, I sent the authenticated request to Burp Intruder.

The goal was to recreate valid stay-logged-in cookies automatically.

---

## Screenshot 7 — Verifying the Cookie Logic

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Brute-Forcing%20a%20Stay-Logged-In%20Cookie/screenshots/lab9(7).png?raw=true)

I modified the request:

```http
GET /my-account
```

and removed unnecessary headers such as:

```http
Referer
```

I then loaded:

```text
peter
```

into the payload section.

Using **Grep Match**, I added:

```text
Update Email
```

This would help identify valid authenticated responses.

---

## Screenshot 8 — Payload Processing Rules

![Screenshot 8](screenshot-auth8.png)

I configured Intruder Payload Processing with:

```text
Hash → MD5
Add Prefix → wiener:
Base64 Encode
```

---

# 🔍 Understanding Payload Processing

Burp automatically transformed:

```text
peter
```

into:

### Step 1

```text
MD5(peter)
```

↓

```text
51dc30ddc474d43a6011e9ebba6ca770
```

### Step 2

```text
wiener:51dc30ddc474d43a6011e9ebba6ca770
```

### Step 3

```text
Base64 Encode
```

↓

```text
d2llbmVyOjUxZGMzMGRkYzQ3NGQ0M2E2MDExZTllYmJhNmNhNzcw
```

This recreated the exact cookie generated by the application.

---

## Screenshot 9 — Confirming the Logic

![Screenshot 9](screenshot-auth9.png)

After running the attack, I sorted the results using the:

```text
Update Email
```

column.

A valid response appeared.

Rendering the response showed:

```text
Username: wiener
```

and

```text
Update Email
```

confirming that the cookie generation logic was correct.

---

## Screenshot 10 — Brute Forcing Carlos's Password

![Screenshot 10](screenshot-auth10.png)

I then replaced the payload list with the password list provided by the lab.

The payload processing rules became:

```text
Hash → MD5
Add Prefix → carlos:
Base64 Encode
```

Grep Match:

```text
Update Email
```

The attack was then launched.

---

## Screenshot 11 — Discovering Carlos's Cookie

![Screenshot 11](screenshot-auth11.png)

After sorting the results, one response contained:

```text
Update Email
```

indicating successful authentication.

Rendering the response showed:

```text
Username: carlos
```

which confirmed that a valid stay-logged-in cookie had been generated.

---

## Screenshot 12 — Request in Browser

![Screenshot 12](screenshot-auth12.png)

Using Burp Suite's:

```text
Request in Browser
```

feature, I copied the successful authenticated request.

---

## Screenshot 13 — Successful Login as Carlos

![Screenshot 13](screenshot-auth13.png)

After opening the request in the browser, I was successfully authenticated as:

```text
carlos
```

The lab was solved.

---

# 🔄 Attack Flow

```text
Login as Wiener
        │
        ▼
Capture Stay Logged In Cookie
        │
        ▼
Decode Base64
        │
        ▼
Identify MD5 Hash
        │
        ▼
Crack Password Hash
        │
        ▼
Discover Cookie Format
        │
        ▼
Generate Carlos Cookies
        │
        ▼
Brute Force Password List
        │
        ▼
Find Valid Cookie
        │
        ▼
Login as Carlos
```

---

# ⚠ Why the Attack Worked

The attack succeeded because:

- authentication tokens were predictable
- the cookie contained credential-derived data
- MD5 hashes were used
- no server-side token validation existed
- attackers could recreate cookies offline

The application trusted user-controlled data for authentication.

---

# 💥 Impact

An attacker could potentially:

- impersonate users
- bypass authentication
- hijack accounts
- gain unauthorized access
- perform privilege escalation
- compromise sensitive user data

If administrator accounts used the same mechanism, full application compromise could occur.

---

# 🛡 Mitigation

To prevent this issue:

### Use Random Session Tokens

```text
stay-logged-in=random_token
```

instead of credential-derived values.

### Store Tokens Server-Side

Maintain token validation on the server.

### Avoid MD5

Use:

- bcrypt
- Argon2
- PBKDF2

for password storage.

### Rotate Persistent Login Tokens

Generate new tokens after:

- login
- password changes
- logout

### Implement Token Expiration

Persistent tokens should have limited lifetimes.

### Monitor Authentication Anomalies

Detect unusual cookie usage patterns.

---

# 🧠 Skills Learned

- Authentication Testing
- Cookie Analysis
- Base64 Decoding
- MD5 Hash Identification
- Password Hash Cracking
- Burp Intruder Payload Processing
- Persistent Login Testing
- Account Takeover Techniques

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- Burp Inspector
- CrackStation
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how an insecure stay-logged-in implementation can completely undermine authentication security.

By analyzing the cookie structure, decoding the Base64 value, identifying the embedded MD5 hash, and understanding how the application generated authentication tokens, I was able to recreate valid cookies for another user and gain unauthorized access to Carlos's account.

Through this lab, I learned:

- how persistent login cookies work
- why credential-derived tokens are dangerous
- how Burp Intruder payload processing can automate token generation
- why MD5 should never be used for authentication mechanisms

The lab was successfully solved by reverse engineering the stay-logged-in cookie format and generating a valid authentication token for Carlos.
