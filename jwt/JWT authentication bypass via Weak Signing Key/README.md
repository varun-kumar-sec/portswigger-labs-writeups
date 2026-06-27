# JWT Authentication Bypass via Weak Signing Key

## 📖 Lab Overview

### 🔹 What is this lab trying to teach?

This lab demonstrates a common JWT implementation vulnerability where the application signs JWT tokens using a **weak secret key**. Instead of using a long, randomly generated secret, the server uses a predictable password-like key that can be cracked through a dictionary attack.

Once an attacker discovers the signing key, they can:

- Forge valid JWT tokens.
- Modify the token payload.
- Impersonate any user, including the administrator.
- Gain unauthorized access to privileged functionality.

In this lab, the objective is to recover the weak signing key, modify the JWT so that it identifies the user as **administrator**, sign it using the recovered secret, and eventually gain access to the administrator panel.

---

# 🚀 Step-by-Step Walkthrough

## 📷 Screenshot 1

![Screenshot 1](images/screenshot1.png)

The lab starts with a normal shopping website. On the top navigation bar, there is a **My Account** button that allows users to authenticate themselves.

Since JWT tokens are generated only after successful authentication, the first step is to log in using the provided user credentials.

---

## 📷 Screenshot 2

![Screenshot 2](images/screenshot2.png)

After clicking the **My Account** button, I was redirected to the login page.

Using the credentials provided by the lab:

- **Username:** `wiener`
- **Password:** `peter`

I entered both credentials and clicked the **Log in** button.

After successful authentication, the application generated a JWT token that would be used for authorizing future requests.

---

## 📷 Screenshot 3

![Screenshot 3](images/screenshot3.png)

After logging in successfully, I was redirected to my account page.

The URL was:

```text
https://0a5a004f0364f9df80b1626000320022.web-security-academy.net/my-account?id=wiener
```

This confirms that I am currently authenticated as the **wiener** user.

Since the objective of the lab is to gain administrator privileges, the next step is to test whether the administrator panel is accessible.

---

## 📷 Screenshot 4

![Screenshot 4](images/screenshot4.png)

I manually modified the current URL from:

```text
/my-account?id=wiener
```

to

```text
/admin
```

After refreshing the page, the application responded with the following message:

```text
Admin interface only available if logged in as administrator
```

This tells us three important things:

- ✅ The administrator panel exists.
- ✅ My current account is not an administrator.
- ✅ The application is checking my privileges before granting access.

This suggests that the JWT token likely contains information about the currently authenticated user. Therefore, the next step is to inspect the JWT.

---

## 📷 Screenshot 5

![Screenshot 5](images/screenshot5.png)

I captured the **GET /admin** request in Burp Suite and sent it to **Repeater**.

The request contained a JWT token inside the request headers.

After sending the request, the server returned:

```http
HTTP/2 401 Unauthorized
```

This confirms that the current JWT does not provide administrator privileges.

### 🔍 Decoding the JWT

A JWT consists of three Base64URL-encoded sections separated by dots:

```text
Header.Payload.Signature
```

The decoded token contains the following information.

### 📌 Header

```json
{
    "alg": "HS256",
    "kid": "2752c78b-2c6a-42f7-8033-aa50605804d9"
}
```

**Explanation**

- **alg** → Specifies the signing algorithm used to protect the JWT. In this case, the application uses **HS256 (HMAC with SHA-256)**.

- **kid** → Represents the **Key ID**, which helps the server identify which secret key should be used to verify the JWT signature.

### 📌 Payload

```json
{
    "iss": "portswigger",
    "exp": 1782113329,
    "sub": "wiener"
}
```

**Explanation**

- **iss (Issuer)** → Identifies the application that generated the JWT.

- **exp (Expiration Time)** → Specifies the Unix timestamp after which the token becomes invalid.

- **sub (Subject)** → Represents the identity of the authenticated user. In this case, the logged-in user is **wiener**.

From the payload, it is clear that the application determines the logged-in user based on the value of the **sub** claim. If we could modify this claim to **administrator** and generate a valid signature, the server would likely treat us as an administrator.

However, to generate a valid signature, we first need to recover the secret key used by the server.

---

## 📷 Screenshot 6

![Screenshot 6](images/screenshot6.png)

To recover the JWT signing secret, I used **Hashcat** to perform a dictionary attack against the token.

The command used was:

```bash
hashcat -a 0 -m 16500 <JWT_TOKEN> /usr/share/seclists/Passwords/Common-Credentials/darkweb2017_top-10000.txt
```

### 🔎 Command Breakdown

#### ▶️ `hashcat`

Starts the Hashcat password recovery tool.

---

#### ▶️ `-a 0`

Specifies **Attack Mode 0**, which performs a **straight dictionary attack**.

Instead of generating random passwords, Hashcat simply tests every password contained inside the supplied wordlist.

---

#### ▶️ `-m 16500`

Specifies the hash mode.

Hash mode **16500** tells Hashcat that the supplied hash is an **HS256 JSON Web Token (JWT)**.

Hashcat automatically calculates the HMAC-SHA256 signature for every password in the wordlist until it finds a matching signature.

---

#### ▶️ `<JWT_TOKEN>`

This is the JWT whose signing secret needs to be recovered.

---

#### ▶️ `/usr/share/seclists/Passwords/Common-Credentials/darkweb2017_top-10000.txt`

This is the password wordlist.

It contains thousands of commonly used passwords that Hashcat tests one by one against the JWT signature.

Since the application uses a weak signing key, the correct secret is successfully discovered.

---

## 📷 Screenshot 7

![Screenshot 7](images/screenshot7.png)

After Hashcat completed the attack, it successfully recovered the JWT signing secret.

The recovered secret was:

```text
secret1
```

Recovering this secret is the most important part of the attack.

Now that we know the server's signing key, we can generate completely valid JWT tokens that the server will trust.

The next step is to use this recovered secret to create our own administrator token.

---

## 📷 Screenshot 8

![Screenshot 8](images/screenshot8.png)

I returned to Burp Suite and opened the **Decoder** tool.

Since Burp Suite expects the secret in **Base64** format while creating a signing key, I encoded the recovered secret.

Original secret:

```text
secret1
```

Base64 encoded value:

```text
c2VjcmV0MQ==
```

This encoded value will be used while generating the symmetric signing key inside Burp Suite.

---

## 📷 Screenshot 9

![Screenshot 9](images/screenshot9.png)

Next, I opened the **JWT Editor** inside Burp Suite and selected **New Symmetric Key**.

Burp Suite generated a new symmetric key template.

I modified only the value of the **k** parameter by replacing it with the Base64-encoded version of the recovered secret.

```json
{
    "kty": "oct",
    "kid": "d514a66b-fa83-4117-98ab-97ab6237a236",
    "k": "c2VjcmV0MQ=="
}
```

The important modification is:

```json
"k": "c2VjcmV0MQ=="
```

which represents the Base64 encoding of:

```text
secret1
```

After making the modification, I clicked **OK**.

Burp Suite now stores the recovered signing secret and will use it later to generate a valid signature after modifying the JWT payload from **wiener** to **administrator**.

## 📷 Screenshot 10

![Screenshot 10](images/screenshot10.png)

I returned to **Repeater** and clicked the **JSON Web Token** option available above the request. Burp Suite automatically decoded the JWT into its **Header** and **Payload** sections.

### 📌 Header

```json
{
    "kid": "2752c78b-2c6a-42f7-8033-aa50605804d9",
    "alg": "HS256"
}
```

### 📌 Payload

```json
{
    "iss": "portswigger",
    "exp": 1782113329,
    "sub": "wiener"
}
```

The important field in the header is:

```json
"alg": "HS256"
```

### 🔍 What is HS256?

**HS256 (HMAC SHA-256)** is a **symmetric cryptographic signing algorithm** used to protect JWT tokens.

Unlike asymmetric algorithms (such as RS256), **HS256 uses the same secret key for both signing and verifying the JWT**.

The process works as follows:

1. The server creates the JWT Header and Payload.
2. It combines them together.
3. It signs the combined data using the secret key and the **SHA-256 HMAC** algorithm.
4. The generated signature becomes the third part of the JWT.

Whenever the client sends the JWT back to the server, the server repeats the signing process using its own secret key.

- If the generated signature matches the signature inside the JWT, the token is considered valid.
- If the signatures do not match, the request is rejected.

Since I had already recovered the secret key (`secret1`) in the previous steps, I was now able to generate a completely valid signature for any modified JWT.

---

## 📷 Screenshot 11

![Screenshot 11](images/screenshot11.png)

Next, I modified the JWT **Payload**.

The original payload contained:

```json
{
    "iss": "portswigger",
    "exp": 1782113329,
    "sub": "wiener"
}
```

I changed only the **sub (Subject)** claim.

From:

```json
"sub": "wiener"
```

To:

```json
"sub": "administrator"
```

The **sub** claim represents the identity of the authenticated user.

By changing its value from **wiener** to **administrator**, I attempted to impersonate the administrator account.

However, modifying the payload invalidates the existing JWT signature, so the token must be signed again using the recovered secret key.

---

## 📷 Screenshot 12

![Screenshot 12](images/screenshot12.png)

After modifying the payload, I clicked the **Sign** button in Burp Suite.

Burp Suite displayed the previously created symmetric signing key that contained the recovered secret.

I selected that key and clicked **OK**.

Burp Suite automatically generated a **new HS256 signature** using the recovered secret (`secret1`) and updated the JWT.

As a result, the modified token was now cryptographically valid and could be accepted by the server.

---

## 📷 Screenshot 13

![Screenshot 13](images/screenshot13.png)

I sent the request containing the newly signed JWT.

This time, instead of returning **401 Unauthorized**, the server responded with:

```http
HTTP/2 200 OK
```

This confirms that the server successfully verified the JWT signature and trusted the modified token.

Since the **sub** claim now contained **administrator**, the application treated me as an administrator and granted access to the administrator panel.

After scrolling through the response, I found the following endpoint for deleting users:

```text
/admin/delete?username=carlos
```

The lab specifically requires deleting the **carlos** user, so I copied this endpoint for the next step.

---

## 📷 Screenshot 14

![Screenshot 14](images/screenshot14.png)

I modified the request path to:

```http
GET /admin/delete?username=carlos
```

and sent the request.

The server responded with:

```http
HTTP/2 302 Found
```

A **302 Found** response indicates that the requested action was successfully processed, but the server wants the client to visit another page.

To continue following the application's normal workflow, I clicked **Follow Redirection** in Burp Suite.

---

## 📷 Screenshot 15

![Screenshot 15](images/screenshot15.png)

After following the redirection, Burp Suite automatically sent the next request.

This time the server returned:

```http
HTTP/2 200 OK
```

The successful response indicates that the deletion request completed successfully.

---

## 📷 Screenshot 16

![Screenshot 16](images/screenshot16.png)

To view the response in a more user-friendly format, I used Burp Suite's **Show response in browser** feature.

This feature generates a temporary URL that renders the intercepted HTTP response directly inside a web browser.

I copied the generated URL.

---

## 📷 Screenshot 17

![Screenshot 17](images/screenshot17.png)

Finally, I pasted the copied URL into the browser and pressed **Enter**.

After the page loaded, I observed the following:

- ✅ The **carlos** user was no longer present.
- ✅ A message stating **"User deleted successfully"** was displayed.
- ✅ The lab displayed **"Congratulations, you solved the lab!"**

This confirms that forging a valid JWT using the recovered weak signing key successfully granted administrator privileges, allowing unauthorized access to the administrator panel and enabling deletion of the target user account.

---

# 🛡️ Mitigation

To prevent JWT authentication bypass through weak signing keys, developers should follow these security best practices:

- **Use strong, randomly generated signing secrets** (at least 256-bit entropy for HS256).
- **Never use common words, default passwords, or predictable secrets** such as `secret`, `password123`, or `secret1`.
- Store signing keys securely using **environment variables** or a dedicated **secret management solution** instead of hardcoding them into the application.
- Rotate signing keys periodically and immediately after any suspected compromise.
- Validate every JWT signature before trusting its contents.
- Restrict access based on server-side authorization checks rather than relying solely on JWT claims.
- Monitor authentication logs for abnormal JWT usage or repeated invalid signature attempts.

---

# 🎯 Why This Attack Worked

This attack succeeded because the application was using a **weak HMAC signing secret** for its JWT tokens.

The overall attack flow was:

1. Capture a valid JWT belonging to a normal user (`wiener`).
2. Perform an offline brute-force attack using **Hashcat** and a common password wordlist.
3. Recover the signing secret (`secret1`).
4. Import the recovered secret into Burp Suite as a symmetric key.
5. Modify the JWT payload by changing the `sub` claim from `wiener` to `administrator`.
6. Re-sign the modified token using the recovered secret.
7. Since the new signature was cryptographically valid, the server trusted the modified JWT and granted administrator privileges.

In short, **the vulnerability was not in JWT itself—it was caused by using a weak signing key that could be guessed through brute-force attacks.**

---

# 📚 Learning Outcomes

From this lab, I learned:

- How JWTs signed with **HS256** depend entirely on the secrecy of the signing key.
- Why weak signing secrets make JWT authentication vulnerable to offline brute-force attacks.
- How to identify and capture JWTs using Burp Suite.
- How to use **Hashcat** to recover weak JWT secrets.
- How to convert a recovered secret into Base64 format for use in Burp Suite.
- How to generate a symmetric signing key inside Burp Suite.
- How to modify JWT payload claims and generate a valid signature.
- Why servers trust modified JWTs when the signature is valid.
- The importance of choosing strong cryptographic secrets for authentication systems.

---

# ✅ Conclusion

In this lab, I demonstrated how a JWT implementation becomes vulnerable when it relies on a **weak HMAC signing secret**. By capturing a legitimate JWT, brute-forcing the signing key using Hashcat, and re-signing a modified token with administrator privileges, I was able to bypass authentication and gain unauthorized access to the administrator panel. Finally, I used the newly acquired administrative privileges to delete the target user's account, successfully completing the lab.

This lab highlights that **JWT security is only as strong as its signing key**. Even when the JWT implementation is technically correct, using a weak secret completely undermines its security, allowing attackers to forge valid tokens and impersonate privileged users.
