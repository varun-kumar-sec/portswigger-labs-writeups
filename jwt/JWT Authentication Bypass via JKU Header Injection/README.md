# 🧪 Lab: JWT Authentication Bypass via JKU Header Injection

## 📖 Overview

### What is a JKU Header?

The **JKU (JSON Web Key Set URL)** header is an optional JWT header parameter that tells the server **where it should fetch the public key** required to verify the JWT signature.

Unlike the **`jwk`** header, which embeds the public key directly inside the JWT, the **`jku`** header only contains a **URL** that points to a **JSON Web Key Set (JWKS)** file.

A normal JWT header using a JKU looks like:

```json
{
    "alg": "RS256",
    "kid": "12345",
    "jku": "https://example.com/.well-known/jwks.json"
}
```

When the server receives this JWT, it performs the following steps:

1. Reads the **jku** URL.
2. Sends an HTTP request to that URL.
3. Downloads the JSON Web Key Set (JWKS).
4. Finds the matching key using the **kid** value.
5. Uses that public key to verify the JWT signature.

This feature is useful when applications need to rotate public keys without changing server configurations.

---

### ⚠️ Where is the Vulnerability?

The vulnerability occurs when the application **blindly trusts any URL supplied in the `jku` header**.

Instead of downloading keys only from a trusted domain, the server allows attackers to specify **their own URL**.

This means an attacker can:

- Generate their own RSA key pair.
- Host the corresponding public key on a server they control.
- Modify the JWT claims (for example, changing `sub` from `wiener` to `administrator`).
- Sign the JWT using their own private key.
- Point the **jku** header to their malicious public key.

If the application downloads and trusts that key, it successfully verifies the forged JWT and grants administrator access.

---

# 🎯 Objective

The goal of this lab is to exploit a **JKU Header Injection** vulnerability by making the server download an attacker-controlled public key from the exploit server. Using this key, we will forge a valid administrator JWT and gain access to the admin panel.

---

# 🛠️ Steps

---

## 📷 Step 1: Access the Lab

The lab initially loads a normal shopping website containing various products and a **My Account** button in the navigation bar.

No authentication has been performed yet.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(1).png?raw=true)

---

## 📷 Step 2: Login as a Normal User

Click the **My Account** button.

The application redirects to the login page.

Login using the provided credentials:

- **Username:** `wiener`
- **Password:** `peter`

After entering the credentials, click **Log in**.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(2).png?raw=true)

---

## 📷 Step 3: Successfully Logged In

After successful authentication, the application redirects to the user's profile page.

Current URL:

```text
/my-account?id=wiener
```

This confirms that the current authenticated user is **wiener**.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(3).png?raw=true)

---

## 📷 Step 4: Attempt to Access the Administrator Panel

Since administrator interfaces are commonly located under predictable paths, manually modify the URL to:

```text
/admin
```

After refreshing the page, the application displays:

```text
Admin interface only available if logged in as administrator
```

This confirms that authorization is being enforced using the information contained inside the JWT.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(4).png?raw=true)

---

## 📷 Step 5: Capture and Analyze the JWT

Capture the previous request in **Burp Suite Repeater**.

Request:

```http
GET /admin
```

Inside the request headers, a JWT token is present.

After sending the request, the server returns:

```text
401 Unauthorized
```

because the current JWT belongs to a normal user.

### 🔍 Decoding the JWT

A JWT consists of three parts:

```text
Header.Payload.Signature
```

### Header

```json
{
    "kid": "42198812-7f85-4512-9b01-7d723c2473bc",
    "alg": "RS256"
}
```

**Explanation**

- **kid** → Key Identifier. It tells the server which public key should be used for signature verification.
- **alg** → Signing algorithm.

The presence of:

```json
"alg": "RS256"
```

indicates that the application uses **RSA asymmetric cryptography**.

---

### Payload

```json
{
    "iss": "portswigger",
    "exp": 1782448321,
    "sub": "wiener"
}
```

**Explanation**

| Claim | Description |
|--------|-------------|
| **iss** | Token issuer. |
| **exp** | Expiration timestamp. |
| **sub** | Authenticated user. |

The important claim is:

```json
"sub": "wiener"
```

which identifies the currently logged-in user.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(5).png?raw=true)

---

## 📷 Step 6: Generate an RSA Key Pair

Open Burp Suite's **JWT Editor**.

Since the JWT uses **RS256**, generate an RSA key pair by selecting:

```
New RSA Key
```

Choose:

```
Format → JWK
```

Then click:

```
Generate
```

Finally click **OK**.

Burp generates a brand-new RSA key pair.

Next, right-click the generated key and choose:

```
Copy Public Key as JWK
```

### 💡 Why are we generating our own RSA key?

Since the server will eventually verify the JWT using a public key, we need our own RSA key pair.

Later in the attack:

- We will sign the forged JWT using our **private key**.
- The application will verify it using our **public key**.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(6).png?raw=true)

---

## 📷 Step 7: Prepare the JSON Web Key Set (JWKS)

Paste the copied public key into a text editor.

Format it as a proper **JWKS (JSON Web Key Set)**:

```json
{
    "keys": [
        {
            "kty": "RSA",
            "e": "AQAB",
            "kid": "babb072f-ee7c-4b7e-92fd-1fd186166855",
            "n": "..."
        }
    ]
}
```

### 🔍 Understanding the Fields

| Field | Purpose |
|--------|----------|
| **kty** | Key type (RSA). |
| **e** | RSA public exponent. |
| **kid** | Key Identifier used to match this key. |
| **n** | RSA public modulus. |

This JSON file represents the **public key** that the vulnerable server will later download.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(7).png?raw=true)

---

## 📷 Step 8: Host the Public Key on the Exploit Server

Click the **Go to Exploit Server** button provided in the lab.

Paste the entire JWKS JSON into the response body.

Click:

```
Store
```

### 💡 Why are we hosting this file?

The attack relies on making the vulnerable application **download our public key**.

Instead of embedding the key inside the JWT, we place it on the exploit server.

Later, the **jku** header will point to this URL.

When the application receives the JWT, it will:

1. Read the **jku** URL.
2. Visit our exploit server.
3. Download our public key.
4. Use our key to verify our forged JWT.

In other words, we are replacing the server's trusted public key with one that we completely control.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(8).png?raw=true)

---

## 📷 Step 9: Open the JWT Editor

Return to Burp Repeater.

Click the **JSON Web Token** tab available above the request.

Burp automatically decodes both the JWT Header and Payload, making them easy to edit before performing the attack.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(9).png?raw=true)

---

## 📷 Step 10: Modify the JWT Header and Payload

Inside the JWT Editor, modify the **Header** by adding a new parameter called **`jku`**.

Original Header:

```json
{
    "kid": "42198812-7f85-4512-9b01-7d723c2473bc",
    "alg": "RS256"
}
```

Modified Header:

```json
{
    "kid": "42198812-7f85-4512-9b01-7d723c2473bc",
    "alg": "RS256",
    "jku": "https://exploit-0a130086035949f680ec89.exploit-server.net/exploit"
}
```

Next, modify the JWT Payload.

Original:

```json
{
    "iss": "portswigger",
    "exp": 1782448321,
    "sub": "wiener"
}
```

Modified:

```json
{
    "iss": "portswigger",
    "exp": 1782448321,
    "sub": "administrator"
}
```

### 💡 Why add the `jku` header?

Normally, the application verifies JWT signatures using a trusted public key stored on the server.

By adding the **`jku`** header, we are instructing the application to **download the public key from our exploit server instead**.

Since we generated the corresponding private key earlier, we can create a JWT that will successfully verify using our attacker-controlled public key.

At the same time, changing:

```json
"sub": "wiener"
```

to

```json
"sub": "administrator"
```

changes the identity stored inside the JWT.

If the signature verification succeeds, the server will treat us as the administrator.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(10).png?raw=true)

---

## 📷 Step 11: Update the Key Identifier (kid)

The JWT Header still contains the original application's **Key Identifier**.

Replace it with the **`kid`** value from the RSA key we generated earlier.

Original:

```json
"kid":"42198812-7f85-4512-9b01-7d723c2473bc"
```

Modified:

```json
"kid":"babb072f-ee7c-4b7e-92fd-1fd186166855"
```

### 💡 Why is this required?

The exploit server hosts a **JWKS** file containing our public key.

Inside that JWKS, the key is identified using:

```json
"kid":"babb072f-ee7c-4b7e-92fd-1fd186166855"
```

When the server downloads the JWKS, it searches for a key whose **kid** matches the JWT Header.

If these values differ, the server cannot locate the correct public key and signature verification fails.

Updating the **kid** ensures that the application selects our public key.

![](11)

---

## 📷 Step 12: Sign the Modified JWT

Click:

```
Sign
```

Select the RSA key that was generated earlier.

Press:

```
OK
```

Burp automatically signs the modified JWT using the attacker's private key.

The resulting JWT now contains:

- Modified administrator payload
- JKU header
- Matching Key Identifier
- Fresh RSA signature

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(12).png?raw=true)

---

## 📷 Step 13: Verify the JWT

Before sending the request, Burp verifies the newly created signature.

The message displayed is:

```
JWS verified OK
```

This confirms:

- The JWT has been signed correctly.
- The signature matches the embedded RSA key.
- The token is ready to be sent.

Send the modified request.

This time, the response returns:

```
200 OK
```

instead of **401 Unauthorized**.

Scroll through the administrator page until the following endpoint is found:

```text
/admin/delete?username=carlos
```

Copy this endpoint.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(13).png?raw=true)

---

## 📷 Step 14: Delete Carlos

Modify the request:

```http
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

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(14).png?raw=true)

---

## 📷 Step 15: Follow the Redirect

After following the redirect, Burp returns:

```
200 OK
```

This indicates that the delete operation completed successfully.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(15).png?raw=true)

---

## 📷 Step 16: View the Response in Browser

Use Burp Suite's feature:

```
Show response in browser
```

Burp generates a temporary URL.

Copy the generated URL.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(16).png?raw=true)

---

## 📷 Step 17: Confirm the Result

Paste the copied URL into the browser.

The page displays:

```
User has been deleted successfully
```

confirming that Carlos' account has been deleted and administrator privileges were successfully obtained.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(17).png?raw=true)

---

## 📷 Step 18: Lab Solved

The lab displays the success message indicating that the objective has been completed successfully.

The attack successfully abused the **JKU Header Injection** vulnerability to forge a valid administrator JWT and gain access to protected functionality.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20JKU%20Header%20Injection/screenshots/lab5(18).png?raw=true)

---

# 🛡️ Mitigation

To prevent JKU Header Injection vulnerabilities:

- Never trust client-supplied **`jku`** headers.
- Only allow public keys to be downloaded from trusted, whitelisted domains.
- Reject unexpected JKU URLs.
- Validate TLS certificates when downloading JWKS files.
- Ignore JWT headers that attempt to override server-side key configuration.
- Use fixed server-side public keys whenever possible.
- Validate both the **kid** and the origin of downloaded keys.

---

# 💡 Why This Attack Worked

The vulnerability existed because the application **trusted the `jku` header supplied by the client**.

Instead of verifying JWT signatures using its own trusted public key, the application:

1. Read the attacker-controlled **`jku`** URL.
2. Downloaded the public key from the exploit server.
3. Located the matching key using the supplied **kid**.
4. Verified the forged JWT using the attacker's public key.
5. Accepted the modified payload where:

```json
"sub":"administrator"
```

As a result, the application believed the attacker was a legitimate administrator.

---

# 🎓 Key Learning

From this lab, I learned:

- How the **JKU** header works in JWT authentication.
- The difference between **JWK** and **JKU** attacks.
- How JSON Web Key Sets (JWKS) are structured.
- Why the **kid** value is important during key selection.
- How Burp Suite's JWT Editor simplifies JWT attacks.
- How to generate RSA key pairs for JWT signing.
- Why applications should never trust externally supplied public keys.

---

# ✅ Conclusion

This lab demonstrated a **JWT Authentication Bypass via JKU Header Injection** vulnerability.

By generating our own RSA key pair, hosting the public key on the exploit server, modifying the JWT to reference our hosted key through the **`jku`** header, updating the **kid** value, and changing the **sub** claim to **administrator**, we successfully forged a valid administrator JWT.

Because the application blindly trusted the attacker-controlled **JKU URL**, it downloaded our public key, verified the forged signature, and granted administrator privileges. This allowed us to access the admin panel and delete Carlos' account, successfully solving the lab.
