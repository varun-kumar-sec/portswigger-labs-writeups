# 🧪 Lab: JWT Authentication Bypass via `kid` Header Path Traversal

## 📖 Overview

### What is the `kid` Header?

The **`kid` (Key ID)** header is an optional parameter in a JWT header that tells the server **which cryptographic key should be used to verify the JWT signature**.

A typical JWT header looks like:

```json
{
    "kid": "8477efc8-0032-46fa-b39a-ae858fbc1bff",
    "alg": "HS256"
}
```

Here:

- **kid** → Identifies the secret key or public key.
- **alg** → Specifies the signing algorithm.

During verification, the application reads the **kid** value and uses it to locate the corresponding key before verifying the JWT signature.

---

### ⚠️ What is `kid` Header Path Traversal?

Some applications incorrectly use the **kid** value directly as a filename or file path.

Instead of safely looking up keys from a predefined list, the server may perform something similar to:

```php
readFile("/keys/" + kid);
```

If user input is not validated, an attacker can supply path traversal sequences such as:

```text
../../../../../../dev/null
```

allowing the application to read unintended files from the filesystem.

If the server uses the contents of those files as the JWT secret, attackers may be able to forge valid JWTs.

---

# 🎯 Objective

The goal of this lab is to exploit a **Path Traversal vulnerability** in the JWT **`kid`** header. By forcing the server to use **`/dev/null`** as the signing key and signing our forged JWT with the same value, we can impersonate the **administrator** and gain access to the admin panel.

---

# 🛠️ Steps

---

## 📷 Step 1: Access the Lab

The lab initially loads a normal shopping website containing several products and a **My Account** button.

At this stage, no user is authenticated.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(1).png?raw=true)

---

## 📷 Step 2: Login as a Normal User

Click **My Account**.

The application redirects to the login page.

Login using the provided credentials:

- **Username:** `wiener`
- **Password:** `peter`

After entering the credentials, click **Log in**.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(2).png?raw=true)

---

## 📷 Step 3: Successfully Logged In

After successful authentication, the application redirects to:

```text
/my-account?id=wiener
```

This confirms that the current authenticated user is **wiener**.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(3).png?raw=true)

---

## 📷 Step 4: Attempt to Access the Administrator Panel

Modify the current URL to:

```text
/admin
```

After refreshing the page, the application displays:

```text
Admin interface only available if logged in as administrator
```

This confirms that authorization is controlled by the JWT.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(4).png?raw=true)

---

## 📷 Step 5: Capture and Decode the JWT

Capture the previous request inside **Burp Suite Repeater**.

Request:

```http
GET /admin
```

Inside the request headers, a JWT token is present.

After sending the request, the server returns:

```text
401 Unauthorized
```

because the JWT belongs to a normal user.

---

### 🔍 Decoding the JWT

A JWT consists of three parts:

```text
Header.Payload.Signature
```

### Header

```json
{
    "kid": "8477efc8-0032-46fa-b39a-ae858fbc1bff",
    "alg": "HS256"
}
```

**Explanation**

| Header | Description |
|---------|-------------|
| **kid** | Identifies which key the server should use to verify the JWT signature. |
| **alg** | Signing algorithm used to create the JWT. |

---

### Payload

```json
{
    "iss": "portswigger",
    "exp": 178211588,
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
"sub":"wiener"
```

which identifies the currently authenticated user.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(5).png?raw=true)

---

## 📷 Step 6: Inspect the JWT Header

Highlight the **Header** portion of the JWT and open Burp's **Inspector** tab.

The decoded header appears as:

```json
{
    "kid":"8477efc8-0032-46fa-b39a-ae858fbc1bff",
    "alg":"HS256"
}
```

### 💡 Why is `HS256` important?

The algorithm:

```json
"alg":"HS256"
```

indicates that the application uses **symmetric cryptography**.

With **HS256**, the **same secret key** is used for:

- Signing the JWT
- Verifying the JWT

Therefore, if we can control which file is used as the secret key, we can generate our own valid administrator token.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(6).png?raw=true)

---

## 📷 Step 7: Generate a Symmetric Key

Open Burp Suite's **JWT Editor**.

Click:

```
New Symmetric Key
```

Then click:

```
Generate
```

After the key is generated, modify the **k** value to:

```json
"k":"AA=="
```

### 💡 What is `AA==`?

`AA==` is the Base64 encoding of a **single NULL byte**.

```
Hex : 00
Base64 : AA==
```

---

### 💡 Why use a NULL byte?

Later in the attack, we will force the server to read:

```text
/dev/null
```

as the signing key.

On Linux systems:

```text
/dev/null
```

is a special device file that always returns **empty data (NULL bytes)**.

By configuring Burp's signing key as the Base64 representation of a NULL byte (`AA==`), we sign the JWT using the same effective value that the server will obtain from `/dev/null`.

As a result, both the client and server use identical signing material, allowing the forged JWT signature to verify successfully.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(7).png?raw=true)

---

## 📷 Step 8: Open the JWT Editor

Return to Burp Repeater.

Click the **JSON Web Token** tab available above the request.

Burp automatically decodes both the Header and Payload, allowing them to be modified before re-signing the JWT.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(8).png?raw=true)

---

## 📷 Step 9: Exploit the `kid` Header

Modify the JWT Header:

```json
{
    "kid":"../../../../../../dev/null",
    "alg":"HS256"
}
```

Also modify the Payload:

```json
{
    "iss":"portswigger",
    "exp":178211588,
    "sub":"administrator"
}
```

Finally click:

```
Sign
```

and select the previously generated symmetric key.

---

### 💡 Breaking Down the Modified `kid`

```text
../../../../../../dev/null
```

| Component | Purpose |
|------------|----------|
| `../` | Moves one directory upward. |
| Multiple `../` | Traverses out of the application's key directory. |
| `/dev/null` | Linux special device file that returns empty data. |

The vulnerable application uses the **kid** value directly as a filesystem path.

Instead of loading the intended secret key, it loads:

```text
/dev/null
```

Since our JWT was signed using the Base64 representation of a NULL byte (`AA==`), the server verifies the signature successfully.

At the same time, we modified:

```json
"sub":"administrator"
```

which changes our identity from **wiener** to **administrator**.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(9).png?raw=true)

---

## 📷 Step 10: Access the Administrator Panel

Send the modified request.

This time the server responds with:

```
200 OK
```

instead of **401 Unauthorized**.

Scroll through the response until the following endpoint is found:

```text
/admin/delete?username=carlos
```

Copy this endpoint.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(10).png?raw=true)

---

## 📷 Step 11: Delete Carlos

Modify the request:

```http
GET /admin/delete?username=carlos
```

Send the request.

The server responds with:

```
302 Found
```

Click:

```
Follow Redirection
```

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(11).png?raw=true)

---

## 📷 Step 12: Follow the Redirect

After following the redirect, Burp returns:

```
200 OK
```

indicating that the delete operation completed successfully.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(12).png?raw=true)

---

## 📷 Step 13: View the Response in Browser

Use Burp Suite's feature:

```
Show response in browser
```

Burp generates a temporary URL.

Copy the generated URL.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(13).png?raw=true)

---

## 📷 Step 14: Confirm the Result

Paste the copied URL into the browser.

The page displays:

```
User deleted successfully!
```

confirming that Carlos' account has been deleted and the lab has been solved.

![](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/jwt/JWT%20Authentication%20Bypass%20via%20%60kid%60%20Header%20Path%20Traversal/screenshots/lab6(14).png?raw=true)

---

# 🛡️ Mitigation

To prevent `kid` Header Path Traversal attacks:

- Never use the `kid` value directly as a filesystem path.
- Maintain a whitelist of valid key identifiers.
- Reject path traversal sequences such as `../`.
- Prevent access to sensitive files like `/dev/null`.
- Validate all JWT header parameters before using them.
- Store signing keys securely instead of loading them dynamically from user input.

---

# 💡 Why This Attack Worked

The application trusted the **`kid`** header supplied by the client.

Instead of validating it against a predefined list of keys, the server interpreted the **kid** value as a filesystem path.

By supplying:

```text
../../../../../../dev/null
```

the application loaded `/dev/null` as the secret key.

Since the forged JWT was signed using the same effective key (NULL byte represented as `AA==`), the signature verification succeeded.

Combined with changing:

```json
"sub":"administrator"
```

the server accepted the JWT as belonging to the administrator, granting access to the admin panel.

---

# 🎓 Key Learning

From this lab, I learned:

- The purpose of the JWT **`kid`** header.
- How insecure key lookup mechanisms introduce path traversal vulnerabilities.
- The difference between **HS256** (symmetric) and **RS256** (asymmetric) JWT algorithms.
- Why Linux special files such as `/dev/null` can sometimes be abused in authentication attacks.
- How Burp Suite's JWT Editor can generate and sign forged JWTs.
- The importance of validating all JWT header parameters.

---

# ✅ Conclusion

This lab demonstrated a **JWT Authentication Bypass via `kid` Header Path Traversal** vulnerability.

By exploiting insecure handling of the **`kid`** header, we forced the application to load `/dev/null` as the signing key. After generating a JWT signed with the corresponding NULL-byte value and modifying the **`sub`** claim to **administrator**, the server successfully verified our forged token and granted administrator privileges.

Using the newly acquired access, we reached the administrator panel, deleted Carlos' account, and successfully solved the lab.
