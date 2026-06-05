# Broken Brute-Force Protection, Multiple Credentials Per Request

## 📌 Lab Overview

This lab demonstrates a **Broken Brute-Force Protection** vulnerability caused by improper handling of JSON input.

The application attempted to prevent brute-force attacks by limiting the number of login attempts. However, the login endpoint accepted credentials in **JSON format**, and the password parameter could be supplied as an **array of values** instead of a single string.

As a result, multiple passwords could be tested within a single request, allowing an attacker to bypass the intended brute-force protection mechanism.

This lab focused on:

- authentication weaknesses
- brute-force protection bypass
- JSON-based attacks
- multiple credential injection
- Burp Repeater testing

---

# 🔍 What is Brute-Force Protection?

Brute-force protection is designed to stop attackers from repeatedly guessing passwords.

Common defenses include:

- account lockouts
- rate limiting
- CAPTCHA challenges
- IP blocking
- MFA enforcement

Example:

```text
Attempt 1 → Incorrect Password
Attempt 2 → Incorrect Password
Attempt 3 → Account Locked
```

The purpose is to limit how many passwords can be tested against an account.

---

# ⚠ Why Was the Protection Broken?

The application expected a request in the following format:

```json
{
    "username":"carlos",
    "password":"secret"
}
```

However, the backend also accepted:

```json
{
    "username":"carlos",
    "password":[
        "123456",
        "password",
        "qwerty",
        "secret"
    ]
}
```

Instead of rejecting the malformed request, the application processed every password inside the array.

This allowed multiple password attempts to be performed in a single request.

As a result:

```text
1 HTTP Request
=
Many Password Attempts
```

which completely bypassed the intended brute-force protection.

---

# 🎯 Objective

The goal of this lab was to:

- analyze the login functionality
- identify weaknesses in the JSON authentication endpoint
- bypass brute-force protections
- discover Carlos's password
- authenticate as Carlos

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-auth1.png)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Testing Login Functionality

![Screenshot 2](screenshot-auth2.png)

After clicking **My Account**, I landed on the login page.

I entered:

```text
Username: carlos
Password: 1234
```

and clicked **Log in**.

---

## Screenshot 3 — Invalid Credentials

![Screenshot 3](screenshot-auth3.png)

The application responded with:

```text
Invalid username or password.
```

This confirmed that the supplied password was incorrect.

---

## Screenshot 4 — Capturing the Login Request

![Screenshot 4](screenshot-auth4.png)

I captured the login request using Burp Suite.

Request:

```http
POST /login
```

Body:

```json
{
    "username":"carlos",
    "password":"1234"
}
```

The interesting observation was that the application accepted credentials in **JSON format** rather than traditional form parameters.

---

## Screenshot 5 — Rendering the Response

![Screenshot 5](screenshot-auth5.png)

I rendered the server response inside Burp Repeater to better understand how the application processed authentication requests.

At this stage, nothing unusual appeared in the response.

---

## Screenshot 6 — Injecting Multiple Passwords

![Screenshot 6](screenshot-auth6.png)

Since the request body was JSON, I modified the password field.

Instead of sending a single password:

```json
{
    "username":"carlos",
    "password":"1234"
}
```

I supplied the entire password list as an array:

```json
{
    "username":"carlos",
    "password":[
        "123456",
        "password",
        "12345678",
        "qwerty",
        "123456789",
        "12345",
        "1234",
        "111111",
        "1234567",
        "dragon",
        "123123",
        "baseball",
        "abc123",
        "football",
        "monkey"
    ]
}
```

### Why Does This Work?

Normally:

```text
1 Request
=
1 Password Attempt
```

However, because the application incorrectly processed arrays, the backend effectively performed:

```text
Password 1
Password 2
Password 3
...
Password N
```

within a single request.

In other words:

```text
1 Request
=
Entire Password Wordlist
```

This bypassed the brute-force protection because only one HTTP request was counted while many passwords were actually tested.

---

# 🔍 Understanding the Vulnerability

The application likely performed logic similar to:

```pseudo
for each password in password_array:
    if password is correct:
        authenticate user
```

Instead of validating:

```pseudo
password must be string
```

the application accepted arrays and processed them internally.

The security control counted:

```text
HTTP Requests
```

but failed to count:

```text
Password Attempts
```

This logical flaw allowed password brute forcing without triggering protections.

---

## Screenshot 7 — Successful Authentication Response

![Screenshot 7](screenshot-auth7.png)

After sending the modified request, the application responded with:

```http
302 Found
```

A redirect after authentication generally indicates:

```text
Login Successful
```

This confirmed that one of the supplied passwords inside the array was valid.

---

## Screenshot 8 — Using Request in Browser

![Screenshot 8](screenshot-auth8.png)

I used Burp Suite's:

```text
Request in Browser
```

feature to copy the authenticated request.

This allowed me to replay the successful session directly in the browser.

---

## Screenshot 9 — Logged in as Carlos

![Screenshot 9](screenshot-auth9.png)

After pasting the authenticated request into the browser, I was successfully logged in as:

```text
carlos
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application failed to properly validate JSON input types.

Instead of enforcing:

```json
"password":"string"
```

the application also accepted:

```json
"password":[...]
```

and processed every value inside the array.

This allowed attackers to perform multiple password guesses within a single request.

---

# 💥 Impact

An attacker could potentially:

- bypass brute-force protections
- test large password lists quickly
- compromise user accounts
- perform credential stuffing attacks
- gain unauthorized access to sensitive data

---

# 🛡 Mitigation

To prevent this issue:

- strictly validate JSON data types
- enforce password fields as strings
- reject arrays and unexpected input structures
- count password attempts rather than HTTP requests
- implement rate limiting
- enforce MFA
- monitor authentication anomalies

Example:

```json
{
    "password":"secret"
}
```

should be accepted, while:

```json
{
    "password":["secret","admin"]
}
```

should be rejected.

---

# 🧠 Skills Learned

- Authentication Testing
- JSON Parameter Manipulation
- Brute-Force Protection Bypass
- Burp Repeater
- Request Analysis
- Login Workflow Analysis
- Input Validation Testing

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improper JSON input validation can completely undermine brute-force protection mechanisms.

By analyzing the authentication request format, identifying that the password parameter accepted arrays, and supplying multiple passwords within a single request, I was able to bypass the intended protection and authenticate as Carlos.

Through this lab, I learned:

- how JSON-based authentication can introduce security flaws
- why input type validation is critical
- how brute-force protections can fail due to logic errors
- how to test authentication endpoints beyond normal user input

The lab was successfully solved by exploiting a JSON parsing weakness that allowed multiple credentials to be processed in a single authentication request.
