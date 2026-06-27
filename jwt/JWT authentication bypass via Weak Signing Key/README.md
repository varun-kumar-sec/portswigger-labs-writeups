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

---
