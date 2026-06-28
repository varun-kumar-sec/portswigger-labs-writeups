# Lab: JWT Authentication Bypass via JWK Header Injection

## 📖 Overview

### What is a JWK Header?

A **JSON Web Key (JWK)** is a JSON-based format used to represent cryptographic keys. These keys are commonly used in JWT implementations that rely on **asymmetric cryptography** (such as **RS256**) for signing and verifying tokens.

Normally, a server stores its own **public/private key pair**:

- The **private key** is used to sign JWTs.
- The **public key** is used to verify JWT signatures.

A JWT can optionally contain a **`jwk` header**. This header allows the sender to embed a **public key directly inside the JWT**.

A secure application should **never trust** a JWK supplied by the client unless it is explicitly expected and validated against a trusted source.

If the server blindly accepts the **embedded JWK** from the incoming token, an attacker can:

1. Generate their own RSA key pair.
2. Modify the JWT payload (for example, changing the user to `administrator`).
3. Sign the JWT using their own private key.
4. Embed the matching public key inside the JWT header.
5. Trick the server into verifying the signature using the attacker-controlled public key.

This completely breaks the authentication mechanism and allows attackers to forge valid administrator tokens.

---

# 🎯 Objective

The objective of this lab is to exploit a **JWK Header Injection** vulnerability by embedding an attacker-controlled public key inside the JWT, allowing the application to trust a forged administrator token and gain access to the admin panel.

---

# 🛠️ Steps

## Step 1: Access the Login Page

The lab starts with a normal shopping application containing a **My Account** button.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(1).png?raw=true)

---

## Step 2: Login as a Normal User

Click **My Account**.

Log in using the provided credentials:

- **Username:** `wiener`
- **Password:** `peter`

After entering the credentials, click **Log in**.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(2).png?raw=true)

---

## Step 3: Successfully Login

After successful authentication, the application redirects to the user's account page.

Current URL:

```
/my-account?id=wiener
```

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(3).png?raw=true)

---

## Step 4: Attempt to Access the Admin Panel

Manually modify the URL to:

```
/admin
```

The application responds with:

```
Admin interface only available if logged in as an administrator
```

This confirms that the current JWT does not provide administrator privileges.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(4).png?raw=true)

---

## Step 5: Capture the JWT

Capture the request for:

```
GET /admin
```

Inside the request, the application sends a JWT.

After forwarding the request to **Repeater**, sending it results in:

```
401 Unauthorized
```

### Decoding the JWT

The JWT consists of three parts:

**Header**

```json
{
    "kid":"24e31c1c-cb57-44c3-8b13-f095bf6518ea",
    "alg":"RS256"
}
```

**Explanation**

- **kid** → Key Identifier used by the server to determine which public key should verify the signature.
- **alg** → RS256 indicates the application is using RSA asymmetric cryptography.

---

**Payload**

```json
{
    "iss":"portswigger",
    "exp":1782448321,
    "sub":"wiener"
}
```

**Explanation**

- **iss** → Token issuer.
- **exp** → Expiration timestamp.
- **sub** → Authenticated user.

Currently:

```
sub = wiener
```

which explains why administrative resources cannot be accessed.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(5).png?raw=true)

---

## Step 6: Identify the Signing Algorithm

Highlight the JWT Header and open Burp's **Inspector**.

The decoded header shows:

```json
{
    "kid":"24e31c1c-cb57-44c3-8b13-f095bf6518ea",
    "alg":"RS256"
}
```

The important observation here is:

```
alg = RS256
```

Since RS256 is an **asymmetric algorithm**, JWTs are signed using a **private key** and verified using the corresponding **public key**.

This makes the application a potential target for **JWK Header Injection** if it trusts embedded public keys.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(6).png?raw=true)

---

## Step 7: Generate Your Own RSA Key Pair

Open **JWT Editor** inside Burp Suite.

Select:

```
New RSA Key
```

Click:

```
Generate
```

Then click **OK**.

Burp creates a completely new RSA public/private key pair which will later be used to sign our forged administrator token.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(7).png?raw=true)

---

## Step 8: Open the JWT Editor

Return to Repeater.

Click the **JSON Web Token** tab available above the request.

Burp automatically decodes both the JWT Header and JWT Payload, making them easier to edit.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(8).png?raw=true)

---

## Step 9: Modify the JWT and Embed the JWK

Inside the Payload, change:

```json
{
    "sub":"wiener"
}
```

to

```json
{
    "sub":"administrator"
}
```

Next, click:

```
Attack
```

Select:

```
Embedded JWK
```

and click **OK**.

### Why Embed the JWK?

Normally, the server verifies JWT signatures using **its own trusted public key**.

However, this vulnerable application incorrectly accepts a **public key supplied inside the JWT Header**.

Burp automatically:

- Signs the modified JWT using the attacker-generated private key.
- Embeds the matching public key inside the JWT Header as the **`jwk`** parameter.

As a result, when the server verifies the JWT, it unknowingly uses the attacker's public key, making the forged administrator token appear completely valid.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(9).png?raw=true)

---

## Step 10: Verify the Embedded JWK

After the attack completes, Burp automatically inserts the generated **JWK** into the JWT Header.

The JWT now contains the attacker's public key.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(10).png?raw=true)

---

## Step 11: Send the Forged JWT

Send the modified request.

The server now returns:

```
200 OK
```

indicating that the forged administrator JWT has been accepted successfully.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(11).png?raw=true)

---

## Step 12: Locate the Delete Function

Scroll through the administrator page.

Locate the delete endpoint for Carlos:

```
/admin/delete?username=carlos
```

Copy this endpoint.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(12).png?raw=true)

---

## Step 13: Delete Carlos

Modify the request:

```
GET /admin/delete?username=carlos
```

Send the request.

The response returns:

```
302 Found
```

Click:

```
Follow Redirection
```

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(13).png?raw=true)

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(14).png?raw=true)
---

## Step 14: Follow the Redirect

After following the redirect, Burp returns:

```
200 OK
```

confirming that the deletion request completed successfully.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(14).png?raw=true)

---

## Step 15: View the Response in Browser

Use Burp Suite's **Show response in browser** feature.

Copy the generated URL.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(15).png?raw=true)

---

## Step 16: Confirm the Result

Paste the generated URL into the browser.

The page displays:

```
User deleted successfully
```

The lab is now successfully solved.

![screenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(16).png?raw=true)

![sceenshot](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JWK%20Header%20Injection/screenshots/lab4(17).png?raw=true)

---

# 🔐 Mitigation

To prevent JWK Header Injection:

- Never trust a JWK supplied by the client.
- Ignore embedded `jwk` headers unless explicitly required.
- Only verify JWTs using trusted server-side keys.
- Restrict accepted algorithms.
- Validate the `kid` against trusted server-side key stores.
- Reject unexpected JWT headers.

---

# ✅ Why This Worked

The vulnerability existed because the application **trusted the public key embedded inside the JWT Header**.

The attack succeeded because:

- We generated our own RSA key pair.
- Modified the JWT payload from `wiener` to `administrator`.
- Signed the JWT using our private key.
- Embedded our public key into the JWT Header.
- The server incorrectly trusted the embedded key and verified the forged signature using it.

As a result, the application believed we were authenticated as the administrator.

---

# 📚 Learning Outcomes

From this lab, I learned:

- How RS256 JWT authentication works.
- The purpose of JWK headers.
- How Burp Suite's JWT Editor simplifies JWT attacks.
- How to generate RSA key pairs.
- How Embedded JWK attacks work.
- Why trusting client-controlled cryptographic keys is dangerous.
- How JWT signature verification should be implemented securely.

---

# 🏁 Conclusion

This lab demonstrated a classic **JWK Header Injection** vulnerability.

Instead of verifying JWT signatures using a trusted server-side public key, the application accepted an attacker-controlled public key embedded inside the JWT itself. By generating our own RSA key pair, modifying the JWT payload to impersonate the administrator, and embedding our public key into the header, we successfully forged a valid administrator token and deleted the target user account.
