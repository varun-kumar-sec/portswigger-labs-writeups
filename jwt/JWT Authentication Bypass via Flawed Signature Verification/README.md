# JWT Authentication Bypass via Flawed Signature Verification

## 📌 Lab Overview

This lab demonstrates a **JWT Authentication Bypass** vulnerability caused by improper signature verification.

The application relied on a JWT (JSON Web Token) to determine whether a user was authenticated and authorized. However, the server trusted the `alg` value supplied inside the JWT header and failed to properly verify the token's signature.

By changing the algorithm from:

```json
"alg":"RS256"
```

to:

```json
"alg":"none"
```

it was possible to completely remove the signature and create a forged JWT that impersonated the administrator account.

This lab focused on:

- JWT Authentication
- JWT Structure
- Signature Verification
- Algorithm Confusion
- `alg:none` Vulnerability
- Authentication Bypass
- Privilege Escalation

---

# 🔍 What Does the Lab Name Mean?

## JWT Authentication Bypass

This means bypassing the application's authentication mechanism by manipulating a JWT token.

Instead of proving our identity legitimately, we alter the token so the application believes we are another user.

Example:

```json
{
  "sub":"wiener"
}
```

becomes:

```json
{
  "sub":"administrator"
}
```

---

## Flawed Signature Verification

A JWT is normally protected using a cryptographic signature.

Typical validation process:

```text
Receive JWT
↓
Verify Signature
↓
Check Claims
↓
Allow Access
```

In this lab the server made a critical mistake:

```text
Receive JWT
↓
Trust "alg":"none"
↓
Skip Signature Verification
↓
Allow Access
```

As a result, anyone could modify the token contents and impersonate any user.

---

# 🔍 What is JWT?

JWT (**JSON Web Token**) is a compact token format used to securely transmit information between two parties.

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

Contains metadata.

Example:

```json
{
  "alg":"RS256",
  "typ":"JWT"
}
```

---

### Payload

Contains user information and claims.

Example:

```json
{
  "sub":"wiener",
  "iss":"portswigger"
}
```

---

### Signature

Used to verify that the token has not been modified.

Example:

```text
RS256(Header + Payload)
```

---

# 🎯 Objective

The goal of this lab was to:

- analyze a JWT token
- identify flawed signature verification
- change the JWT algorithm to `none`
- impersonate the administrator user
- access the admin panel
- delete Carlos's account
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Initial Webpage

![Screenshot 1](screenshot-upload1.png)

The lab initially displayed a normal webpage containing:

- product listings
- a **My Account** button

---

## Screenshot 2 — Login Page

![Screenshot 2](screenshot-upload2.png)

After clicking **My Account**, I was redirected to the login page.

The provided credentials were:

```text
Username: wiener
Password: peter
```

I entered the credentials and logged in.

---

## Screenshot 3 — Attempting to Access the Admin Panel

![Screenshot 3](screenshot-upload3.png)

After successfully logging in as:

```text
wiener
```

I manually modified the URL:

```text
https://0af4003b04239a42821897c4004000c4.web-security-academy.net/admin
```

to attempt direct access to the administrator panel.

The application responded:

```text
Admin interface only available if logged in as an administrator
```

This indicated that authorization was being enforced.

---

## Screenshot 4 — Capturing the Admin Request

![Screenshot 4](screenshot-upload4.png)

I captured the request:

```http
GET /admin
```

and sent it to Burp Repeater.

Inside the request I found a JWT token.

After sending the request, the response returned:

```http
401 Unauthorized
```

This confirmed that my current JWT did not have administrator privileges.

---

## Screenshot 5 — Decoding the JWT

![Screenshot 5](screenshot-upload5.png)

I used the Burp Suite extension:

```text
JSON Web Tokens
```

to decode the JWT.

The extension revealed the JWT structure:

```text
Header
Payload
Signature
```

and displayed the decoded contents.

This allowed me to inspect both the token claims and the signature algorithm being used.

---

## Screenshot 6 — Modifying the JWT Header

![Screenshot 6](screenshot-upload6.png)

I highlighted the JWT header and opened the Inspector tab.

The decoded header contained:

```json
{
  "kid":"04213739-449a-449a-af39-8e2541a485f3",
  "alg":"RS256"
}
```

I modified:

```json
"alg":"RS256"
```

to:

```json
"alg":"none"
```

and clicked:

```text
Apply changes
```

### What Happens When `alg` is Set to `none`?

Normally:

```text
alg = RS256
↓
Signature Required
↓
Server Verifies Signature
↓
Token Trusted
```

However:

```json
"alg":"none"
```

means:

```text
No Signature
```

Historically, some JWT libraries incorrectly trusted this value.

The vulnerable behavior becomes:

```text
Token Received
↓
alg = none
↓
Skip Signature Verification
↓
Accept Token
```

This allows attackers to modify JWT contents without possessing the signing key.

---

## Screenshot 7 — Modifying the JWT Payload

![Screenshot 7](screenshot-upload7.png)

Next, I highlighted the JWT payload and opened the Inspector tab.

The decoded payload contained:

```json
{
  "iss":"portswigger",
  "exp":1782112425,
  "sub":"wiener"
}
```

Where:

| Claim | Meaning |
|---------|---------|
| iss | Issuer |
| exp | Expiration Time |
| sub | Subject (Current User) |

The important field was:

```json
"sub":"wiener"
```

I modified it to:

```json
"sub":"administrator"
```

and clicked:

```text
Apply changes
```

Burp automatically regenerated the JWT.

---

## Screenshot 8 — Removing the Signature

![Screenshot 8](screenshot-upload8.png)

Since the header now specified:

```json
"alg":"none"
```

the signature was no longer required.

I removed the entire signature section.

The JWT became:

```text
Header.Payload.
```

Notice the trailing dot:

```text
xxxxx.yyyyy.
```

This indicates:

```text
No Signature Present
```

I sent the modified request.

This time the response returned:

```http
200 OK
```

The authentication bypass was successful.

---

## Screenshot 9 — Discovering the Delete Endpoint

![Screenshot 9](screenshot-upload9.png)

After gaining access to the administrator panel response, I examined the returned content.

Inside the response I found two users:

```text
wiener
carlos
```

along with their delete paths.

Since the lab specifically required deleting Carlos's account, I copied:

```text
/admin/delete?username=carlos
```

---

## Screenshot 10 — Deleting Carlos

![Screenshot 10](screenshot-upload10.png)

I modified the request:

```http
GET /admin/delete?username=carlos
```

and sent it.

The response returned:

```http
302 Found
```

indicating that the deletion action had completed and the application was redirecting me elsewhere.

---

## Screenshot 11 — Following the Redirect

![Screenshot 11](screenshot-upload11.png)

I clicked:

```text
Follow redirection
```

### What Does "Follow Redirection" Do?

When a server responds with:

```http
301
302
307
308
```

it instructs the browser to visit a new location.

Burp's:

```text
Follow redirection
```

feature automatically sends the next request to the redirected URL.

Result:

```text
Original Request
↓
302 Redirect
↓
Follow Redirect
↓
Final Response
```

After following the redirect, I received:

```http
200 OK
```

---

## Screenshot 12 — Viewing the Response in Browser

![Screenshot 12](screenshot-upload12.png)

I used Burp's feature:

```text
Show response in browser
```

which generated a temporary URL.

This allows Burp responses to be rendered directly inside a browser.

I copied the generated URL.

---

## Screenshot 13 — Lab Solved

![Screenshot 13](screenshot-upload13.png)

After pasting the generated URL into the browser, the page loaded successfully.

I could see:

```text
User deleted successfully
```

and only one user remained:

```text
wiener
```

Carlos's account had been successfully deleted.

The lab was solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- the server trusted the JWT header
- signature verification could be disabled by the client
- the application accepted `alg:none`
- user-controlled claims were trusted without verification

The resulting logic became:

```text
Attacker Modifies JWT
↓
alg = none
↓
Signature Ignored
↓
sub = administrator
↓
Admin Access Granted
```

---

# 💥 Impact

An attacker could potentially:

- bypass authentication
- impersonate other users
- gain administrator access
- access sensitive data
- delete accounts
- perform privileged actions
- completely compromise the application

---

# 🛡 Mitigation

To prevent this vulnerability:

- never allow `alg:none`
- enforce a specific signing algorithm server-side
- ignore client-supplied algorithm values
- always verify JWT signatures
- validate all claims before granting access

Secure validation process:

```text
Receive JWT
↓
Verify Signature
↓
Verify Algorithm
↓
Validate Claims
↓
Grant Access
```

---

# 🧠 Skills Learned

- JWT Structure Analysis
- JWT Header Manipulation
- JWT Payload Manipulation
- Base64 Decoding
- Signature Verification Weaknesses
- Authentication Bypass
- Privilege Escalation
- Burp Suite JWT Testing

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Burp Inspector
- JSON Web Tokens Extension
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improper JWT signature verification can lead to complete authentication bypass.

By changing the JWT algorithm from:

```json
"alg":"RS256"
```

to:

```json
"alg":"none"
```

and modifying the subject claim from:

```json
"sub":"wiener"
```

to:

```json
"sub":"administrator"
```

it was possible to forge a valid administrator token without possessing the server's signing key.

After gaining administrator access, I accessed the admin panel, deleted Carlos's account, and successfully solved the lab.

Through this lab, I learned:

- how JWT authentication works
- why signatures are critical
- how `alg:none` vulnerabilities occur
- how flawed JWT validation leads to authentication bypass
- how JWT claims can be abused for privilege escalation

The lab was successfully solved by exploiting improper JWT signature verification and forging an administrator token.
