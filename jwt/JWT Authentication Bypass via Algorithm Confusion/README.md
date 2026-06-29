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
