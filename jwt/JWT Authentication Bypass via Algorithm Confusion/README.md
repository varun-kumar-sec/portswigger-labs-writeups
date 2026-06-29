# Lab: JWT Authentication Bypass via Algorithm Confusion

## 🎯 Lab Objective

This lab demonstrates an **Algorithm Confusion Attack** against JWT authentication.

Normally, a server signs JWTs using an **asymmetric algorithm** such as **RS256**, where:

- The **private key** signs the token.
- The **public key** verifies the signature.

A vulnerable server incorrectly trusts the **`alg`** value supplied inside the JWT header.

An attacker can exploit this by:

1. Obtaining the application's **public RSA key**.
2. Changing the algorithm from **RS256** to **HS256**.
3. Using the **public key as the HMAC secret** to generate a valid signature.
4. Modifying the JWT payload to impersonate another user (Administrator).

Because the server blindly trusts the algorithm specified in the JWT header, it verifies the forged HS256 signature using the public key as the HMAC secret, allowing authentication bypass.

---

# Step 1: Accessing the Login Page

![]()

The lab starts with the application's home page containing a **My Account** button.

To authenticate, click **My Account**.

---

# Step 2: Logging In

![]()

The application redirects to the login page.

Log in using the provided credentials:

- **Username:** `wiener`
- **Password:** `peter`

After entering the credentials, click **Log in**.

---

# Step 3: Successful Login

![]()

After authentication, the application redirects to:

```text
https://0ad1005d04c99d51807c123400460026.web-security-academy.net/my-account?id=wiener
```

This confirms that we are authenticated as the normal user **wiener**.

---

# Step 4: Attempting to Access the Admin Panel

![]()

Next, manually modify the URL to access the administrator panel.

```text
https://0ad1005d04c99d51807c123400460026.web-security-academy.net/admin
```

The application responds with:

```
Admin interface only available if logged in as administrator
```

This confirms that additional authorization checks are performed.

---

# Step 5: Capturing the JWT

![]()

Capture the previous request in Burp Suite and send it to **Repeater**.

Request:

```http
GET /admin
```

Inside the request is a JWT token.

Example:

```text
eyJ...<JWT>...
```

After sending the request, the response is:

```http
401 Unauthorized
```

---

## 🔍 Decoding the JWT

A JWT consists of three Base64URL-encoded sections.

```
Header.Payload.Signature
```

### Header

```json
{
    "kid": "9fcdad2d-e30a-464f-b0a6-4fe228920952",
    "alg": "RS256"
}
```

**Explanation**

- **kid** → Key Identifier used by the server to determine which key should verify the signature.
- **alg** → Indicates the signing algorithm.

Here,

```
RS256
```

means the application uses:

- RSA Public Key Cryptography
- SHA-256 hashing

This tells us the application signs tokens using a **private RSA key**.

---

### Payload

```json
{
    "iss": "portswigger",
    "exp": 1782113329,
    "sub": "wiener"
}
```

**Explanation**

| Claim | Purpose |
|--------|----------|
| iss | Token issuer |
| exp | Expiration timestamp |
| sub | Identity of the authenticated user |

Currently,

```
sub = wiener
```

meaning this token belongs to the normal user.

---

# Step 6: Accessing the Public JWK Endpoint

![]()

Next, browse to the following endpoint:

```text
https://0ad1005d04c99d51807c123400460026.web-security-academy.net/jwks.json
```

The server exposes a JSON Web Key Set (JWKS).

Response:

```json
{
  "keys": [
    {
      "kty": "RSA",
      "e": "AQAB",
      "use": "sig",
      "kid": "9fcdad2d-e30a-464f-b0a6-4fe228920952",
      "alg": "RS256",
      "n": "l5dU9QuiIu5c9tfRF1g0jlSJ..."
    }
  ]
}
```

---

## 🔍 Why is this important?

The endpoint exposes the application's **public RSA key**.

Normally, publishing the public key is **not a vulnerability** because:

- Public keys are intended to be publicly available.
- They are used only for **verification**, not signing.

However, in an **Algorithm Confusion Attack**, this public key becomes extremely valuable because the attacker can trick the server into treating it as an **HMAC secret**.

This only works when the JWT implementation incorrectly trusts the **`alg`** header.

---

# Step 7: Creating an RSA Key in Burp

![]()

Open **JWT Editor** in Burp Suite.

Click:

```
New RSA Key
```

Since the JWT uses **RS256**, Burp generates an RSA key pair.

Instead of generating random values, replace the generated key with the public key obtained from `jwks.json`.

Paste:

```json
{
  "kty": "RSA",
  "e": "AQAB",
  "use": "sig",
  "kid": "9fcdad2d-e30a-464f-b0a6-4fe228920952",
  "alg": "RS256",
  "n": "l5dU9QuiIu5c9tfRF1g0jlSJ..."
}
```

Click **OK**.

After importing the key:

- Right-click the RSA key.
- Select:

```
Copy Public Key as PEM
```

---

## 🔍 Why convert the public key to PEM?

PEM (Privacy Enhanced Mail) is a standard text-based format used to represent RSA keys.

Example:

```text
-----BEGIN PUBLIC KEY-----
MIIBIjANBg...
-----END PUBLIC KEY-----
```

Later in the attack, this PEM-formatted public key will be Base64 encoded and reused as an **HMAC secret**.

This is the core idea behind the Algorithm Confusion attack:

- The application expects RSA verification.
- We force it to verify an HS256 token.
- The RSA public key becomes the HMAC secret.

---

## 📸 Screenshot 8: Encoding the PEM Public Key into Base64

![Screenshot 8](images/screenshot8.png)

After copying the public key in **PEM** format from the JWT Editor, the next step was to convert it into **Base64** using Burp Suite's **Decoder**.

The copied PEM key looked similar to the following:

```pem
-----BEGIN PUBLIC KEY-----
...
-----END PUBLIC KEY-----
```

This PEM-formatted key was pasted into Burp Suite's **Decoder** tab and encoded into **Base64**.

### 🔍 Why was this required?

The goal of this lab is to perform an **algorithm confusion attack**.

Later, the server will be tricked into using the **public RSA key as an HMAC secret**.

However, Burp's **Symmetric Key Editor** expects the secret to be supplied in **Base64 format**, not raw PEM format.

Therefore:

- Copy the RSA public key as PEM.
- Encode the PEM into Base64.
- Use this Base64 value as the secret (`k`) when generating the symmetric key.

This converts the public RSA key into a format Burp can use for signing an **HS256** JWT.

> **Key idea:**  
> The server already knows the RSA public key because it publishes it. During the algorithm confusion attack, we abuse this by making the server treat that public key as an HMAC secret.

### 🎯 Objective

Prepare the RSA public key in a format that Burp Suite can use to create a forged **HS256-signed JWT** during the next steps.

---

## 🖼️ Screenshot 09: Creating a Symmetric Key from the Public Key

![Screenshot 09](images/screenshot09.png)

After obtaining the server's **public RSA key** and converting it into **Base64**, the next step was to create a **new symmetric key** inside Burp Suite's JWT Editor.

To do this, I clicked **New Symmetric Key** and generated a new key. After the key was created, I replaced the value of the **`k`** parameter with the **Base64-encoded PEM public key** generated in the previous step, and then clicked **OK**.

### 🔍 Why did this work?

This step is the core of the **Algorithm Confusion attack**.

Normally:

- **RS256** uses:
  - A **private key** to sign the JWT.
  - A **public key** to verify the signature.

However, if the server incorrectly accepts **HS256** tokens while still using the **same public RSA key** as the HMAC secret, an attacker can abuse this behavior.

Instead of requiring the private key, the vulnerable server treats the **public key** as a **shared secret**.

Therefore, by creating a symmetric key whose value is exactly the server's public key, Burp can generate a valid **HS256 signature** that the vulnerable server mistakenly trusts.

> In short:
>
> - Server's public key ➜ Converted to Base64
> - Base64 public key ➜ Used as the HMAC secret
> - Burp signs the modified JWT using HS256
> - Vulnerable server verifies the token using the same public key as the secret
> - Signature validation succeeds

---

## 🖼️ Screenshot 10: Modifying and Signing the JWT

![Screenshot 10](images/screenshot10.png)

Next, I opened the **JSON Web Token** editor from the request banner in Burp Suite. Burp automatically decoded both the JWT header and payload.

### Original Header

```json
{
    "kid": "9fcdad2d-e30a-464f-b0a6-4fe228920952",
    "alg": "RS256"
}
```

### Original Payload

```json
{
    "iss": "portswigger",
    "exp": 1782113329,
    "sub": "wiener"
}
```

I then made two important modifications.

### 1. Changed the Algorithm

```json
"alg": "RS256"
```

to

```json
"alg": "HS256"
```

### Why?

The vulnerable server accepts both **RS256** and **HS256**.

By changing the algorithm to **HS256**, the server switches from verifying the signature with RSA to verifying it with HMAC.

Since we already created an HMAC key using the server's public key, we can now generate our own valid signature.

---

### 2. Changed the Subject

```json
"sub": "wiener"
```

to

```json
"sub": "administrator"
```

The **`sub`** claim identifies the authenticated user.

Changing it to **administrator** tells the application that the request belongs to the administrator account.

Finally, I clicked **Sign**, selected the symmetric key created in the previous step, and pressed **OK**.

Burp generated a new HS256 signature using the Base64-encoded public key.

---

## 🖼️ Screenshot 11: Accessing the Administrator Panel

![Screenshot 11](images/screenshot11.png)

After sending the newly signed JWT, the server responded with **200 OK**, indicating that the token had been accepted successfully.

Because the server trusted the forged JWT, it believed that I was authenticated as the **administrator**.

Scrolling through the response revealed several administrative functions, including the following endpoint:

```text
/admin/delete?username=carlos
```

This endpoint allows an administrator to delete user accounts.

Since the lab specifically requires deleting **Carlos**, I copied this endpoint for the next step.

---

## 🖼️ Screenshot 12: Deleting Carlos

![Screenshot 12](images/screenshot12.png)

I modified the request to:

```http
GET /admin/delete?username=carlos
```

After sending the request, the server responded with:

```http
302 Found
```

A **302 Found** response indicates that the deletion request was processed successfully and the application is redirecting the browser to another page.

I clicked **Follow Redirection** to complete the request flow.

---

## 🖼️ Screenshot 13: Following the Redirect

![Screenshot 13](images/screenshot13.png)

After following the redirect, the server returned:

```http
HTTP/1.1 200 OK
```

This confirmed that the deletion operation completed successfully.

---

## 🖼️ Screenshot 14: Viewing the Response in Browser

![Screenshot 14](images/screenshot14.png)

To verify the result visually, I used Burp Suite's **Show Response in Browser** feature.

Burp generated a temporary URL representing the HTTP response.

I copied this generated URL.

---

## 🖼️ Screenshot 15: Lab Successfully Solved

![Screenshot 15](images/screenshot15.png)

Finally, I pasted the copied URL into the browser.

After loading the page, the application displayed the message:

```text
User deleted successfully
```

This confirmed that:

- The forged JWT was accepted.
- Administrator privileges were obtained.
- Carlos' account was successfully deleted.

As a result, the lab was solved successfully.

---

# 💡 Why This Attack Worked

This attack succeeded because the application was vulnerable to an **Algorithm Confusion** flaw.

The server originally signed JWTs using **RS256**, which should require a private key for signing and a public key for verification.

However, the server trusted the **`alg`** value supplied inside the JWT header.

When the attacker changed:

```json
"alg": "RS256"
```

to

```json
"alg": "HS256"
```

the server switched to HMAC verification.

Instead of rejecting the token, it mistakenly reused its own **public RSA key** as the HMAC secret.

Since the public key is publicly accessible through:

```text
/jwks.json
```

the attacker could:

1. Obtain the public key.
2. Convert it into Base64.
3. Use it as the HMAC secret.
4. Generate a valid HS256 signature.
5. Modify the JWT payload to impersonate the administrator.

Because the server incorrectly reused the public key as a symmetric secret, it accepted the forged JWT.

---

# 🛡️ Mitigation

To prevent Algorithm Confusion attacks:

- Never trust the **`alg`** value supplied inside a JWT.
- Explicitly configure the server to accept only the expected algorithm.
- Never mix symmetric (HS256) and asymmetric (RS256) verification logic.
- Separate verification routines for each algorithm.
- Validate the expected key type before verifying signatures.
- Restrict access to sensitive key material and rotate keys periodically.
- Use well-maintained JWT libraries that safely enforce algorithm validation.

---

# 📚 Learning

From this lab, I learned:

- How **Algorithm Confusion** attacks occur.
- The difference between **RS256** and **HS256**.
- Why exposing **JWKS endpoints** can become dangerous when verification is implemented incorrectly.
- How to extract a public RSA key from **JWKS**.
- How to convert a PEM key into Base64.
- How to generate a symmetric key using the server's public key.
- How to modify JWT claims.
- How to re-sign JWTs using Burp Suite's JWT Editor.
- How improper JWT verification can lead to complete privilege escalation.

---

# ✅ Conclusion

In this lab, the application incorrectly trusted the **`alg`** value provided by the client.

By changing the algorithm from **RS256** to **HS256**, using the server's publicly available RSA key as the HMAC secret, and modifying the **`sub`** claim to **administrator**, I successfully forged a valid JWT.

The server accepted the manipulated token, granted administrator privileges, and allowed access to privileged functionality, enabling the deletion of Carlos' account and successfully completing the lab.
