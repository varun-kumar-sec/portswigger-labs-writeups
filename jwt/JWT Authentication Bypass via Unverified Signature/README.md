# JWT Authentication Bypass via Unverified Signature

## 📌 Lab Overview

This lab demonstrates a **JWT Authentication Bypass** vulnerability caused by improper verification of JWT signatures.

The application relied on a JWT token to determine the identity of the logged-in user. However, the server failed to properly validate whether the token's contents had been modified.

As a result, it was possible to:

- modify the JWT payload
- change the user identity from `wiener` to `administrator`
- access administrator-only functionality
- delete Carlos's account
- solve the lab

This lab focused on:

- JWT Authentication
- JWT Structure
- JWT Payload Manipulation
- Signature Validation Issues
- Authorization Bypass
- Privilege Escalation

---

# 🔍 What is JWT?

JWT stands for:

```text
JSON Web Token
```

A JWT is a compact token format commonly used for:

- Authentication
- Authorization
- Session Management

Instead of storing session information on the server, the application stores user information inside a token.

A JWT consists of three parts:

```text
Header.Payload.Signature
```

Example:

```text
xxxxx.yyyyy.zzzzz
```

Where:

### Header

Contains metadata about the token.

Example:

```json
{
  "alg":"RS256",
  "typ":"JWT"
}
```

### Payload

Contains user-related information.

Example:

```json
{
  "sub":"wiener",
  "role":"user"
}
```

### Signature

Used to verify that the token has not been modified.

Example:

```text
Signed(Header + Payload + Secret/Private Key)
```

---

# 🔍 Understanding the Lab Name

## JWT Authentication Bypass

The application uses a JWT token to identify users.

Instead of trusting server-side sessions, it trusts data stored inside the JWT.

---

## Via Unverified Signature

Normally, if an attacker modifies:

```json
{
  "sub":"wiener"
}
```

to:

```json
{
  "sub":"administrator"
}
```

the JWT signature should become invalid.

A secure server would reject the modified token.

However, in this lab:

```text
Signature Validation = Broken
```

The server trusted the modified payload without properly verifying the signature.

This allowed privilege escalation from:

```text
wiener
↓
administrator
```

---

# 🎯 Objective

The goal of this lab was to:

- access the administrator interface
- modify a JWT token
- impersonate administrator
- delete Carlos's account
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Initial Application

![Screenshot 1](screenshot-upload1.png)

The application initially displayed:

- normal webpage
- My Account button

---

## Screenshot 2 — Login

![Screenshot 2](screenshot-upload2.png)

After clicking **My Account**, I was redirected to the login page.

The lab provided credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and logged in.

---

## Screenshot 3 — Attempting to Access Admin Panel

![Screenshot 3](screenshot-upload3.png)

After successfully logging in as:

```text
wiener
```

I manually modified the URL:

```text
https://LAB-ID.web-security-academy.net/admin
```

to attempt access to the administrator interface.

The application responded:

```text
Admin interface only available if logged in as an administrator
```

This confirmed that:

- administrator functionality exists
- access control checks are enforced

---

## Screenshot 4 — Capturing the Request

![Screenshot 4](screenshot-upload4.png)

I captured the request:

```http
GET /admin
```

and sent it to Burp Repeater.

Inside the request, I noticed a JWT token.

Example:

```text
eyJ...Header...
.
eyJ...Payload...
.
bX1...Signature...
```

The server responded:

```text
401 Unauthorized
```

indicating that my current account lacked administrator privileges.

---

## Screenshot 5 — Inspecting the JWT Payload

![Screenshot 5](screenshot-upload5.png)

Using Burp's JWT Inspector, I selected the payload section of the token.

The payload decoded to:

```json
{
  "iss":"portswigger",
  "exp":178211588,
  "sub":"wiener"
}
```

### Breakdown

#### iss

Issuer of the token.

```json
"iss":"portswigger"
```

---

#### exp

Expiration timestamp.

```json
"exp":178211588
```

Defines when the token becomes invalid.

---

#### sub

Subject.

```json
"sub":"wiener"
```

Represents the identity of the authenticated user.

This was the most interesting field because it controlled who the application believed I was.

---

## Screenshot 6 — Modifying the Subject

![Screenshot 6](screenshot-upload6.png)

I changed:

```json
{
  "sub":"wiener"
}
```

to:

```json
{
  "sub":"administrator"
}
```

Resulting payload:

```json
{
  "iss":"portswigger",
  "exp":178211588,
  "sub":"administrator"
}
```

Then I clicked:

```text
Apply Changes
```

---

## Screenshot 7 — Modified JWT Token

![Screenshot 7](screenshot-upload7.png)

After clicking **Apply Changes**, Burp automatically regenerated the payload portion of the JWT.

The modified token now contained:

```json
{
  "sub":"administrator"
}
```

I sent the modified request.

The response returned:

```text
200 OK
```

---

### Why Did This Work?

Normally:

```text
Payload Modification
↓
Signature Becomes Invalid
↓
Server Rejects Token
```

However, in this lab:

```text
Payload Modification
↓
Signature Not Properly Verified
↓
Server Accepts Token
```

The application blindly trusted the modified payload.

This resulted in:

```text
wiener
↓
administrator
```

privilege escalation.

---

## Screenshot 8 — Discovering the Delete Function

![Screenshot 8](screenshot-upload8.png)

After gaining administrator access, I reviewed the response.

Inside the response I found administrative actions including:

```text
/admin/delete?username=wiener

/admin/delete?username=carlos
```

The lab specifically required deleting Carlos's account.

Therefore I copied:

```text
/admin/delete?username=carlos
```

---

## Screenshot 9 — Deleting Carlos

![Screenshot 9](screenshot-upload9.png)

I modified the request:

```http
GET /admin/delete?username=carlos
```

and sent it.

---

## Screenshot 10 — Redirect Response

![Screenshot 10](screenshot-upload10.png)

The server responded:

```http
302 Found
```

which indicates:

```text
Request Processed
↓
Redirect User Somewhere Else
```

At this point I clicked:

```text
Follow Redirection
```

---

### What Does "Follow Redirection" Do?

When a server returns:

```http
301
302
307
308
```

it usually includes a:

```http
Location:
```

header.

Example:

```http
Location: /admin
```

The browser automatically follows that redirect.

Burp's:

```text
Follow Redirection
```

button performs the same action automatically and requests the destination page.

---

## Screenshot 11 — Successful Response

![Screenshot 11](screenshot-upload11.png)

After following the redirect, the server returned:

```http
200 OK
```

indicating that the operation completed successfully.

---

## Screenshot 12 — Viewing the Response in Browser

![Screenshot 12](screenshot-upload12.png)

To view the administrator page more easily, I used Burp's:

```text
Show Response In Browser
```

feature.

Burp generated a temporary URL which I copied.

---

## Screenshot 13 — Confirming Carlos Was Deleted

![Screenshot 13](screenshot-upload13.png)

After opening the generated URL in the browser, I could see that:

```text
wiener
```

was still present, while:

```text
carlos
```

was no longer listed.

This confirmed that Carlos's account had been successfully deleted.

The lab was solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- JWT payloads were trusted blindly
- signature verification was missing or broken
- authorization relied entirely on user-controlled data
- the server accepted modified JWT contents

The flawed logic looked like:

```text
Receive JWT
↓
Read Payload
↓
Trust Payload
↓
Grant Access
```

instead of:

```text
Receive JWT
↓
Verify Signature
↓
Validate Claims
↓
Grant Access
```

---

# 💥 Impact

An attacker could potentially:

- impersonate other users
- become administrator
- bypass authentication
- perform privileged actions
- access sensitive data
- delete accounts
- compromise the entire application

---

# 🛡 Mitigation

To prevent this issue:

- always verify JWT signatures
- reject modified tokens
- validate issuer claims
- validate expiration claims
- validate audience claims
- use secure JWT libraries
- avoid trusting client-controlled claims for authorization

Example:

```text
Receive JWT
↓
Verify Signature
↓
Validate Claims
↓
Authorize User
```

---

# 🧠 Skills Learned

- JWT Fundamentals
- JWT Structure Analysis
- JWT Payload Manipulation
- Authentication Bypass
- Authorization Testing
- Privilege Escalation
- Burp JWT Inspector

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- JWT Inspector
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improper JWT signature validation can completely undermine an application's authentication and authorization mechanisms.

By inspecting the JWT payload, modifying the:

```json
"sub":"wiener"
```

claim to:

```json
"sub":"administrator"
```

and exploiting the application's failure to verify the token signature correctly, I was able to gain administrator privileges.

Once administrative access was obtained, I located the delete-user functionality, removed Carlos's account, and successfully solved the lab.

Through this lab, I learned:

- how JWTs are structured
- how JWT claims control identity
- why signature verification is critical
- how JWT manipulation can lead to privilege escalation
- how broken JWT validation results in authentication bypass

The lab was successfully solved by exploiting a JWT signature verification flaw and impersonating the administrator account.
