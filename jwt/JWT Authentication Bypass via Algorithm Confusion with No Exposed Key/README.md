# Lab: JWT Authentication Bypass via Algorithm Confusion with No Exposed Key

## 🎯 Lab Objective

This lab demonstrates a **JWT Algorithm Confusion attack** in a scenario where the server **does not expose its public key directly**.

Normally, when an application uses **RS256 (asymmetric cryptography)**, the attacker cannot forge JWT tokens because the private key is secret. Even though the public key isn't publicly available through an endpoint like `/jwks.json`, this lab shows that it can still be recovered by analyzing multiple JWT signatures.

After recovering the public key, the attacker abuses an **algorithm confusion vulnerability** by changing the JWT algorithm from **RS256** to **HS256**, using the recovered public key as the HMAC secret. Since the server incorrectly accepts this token, the attacker successfully impersonates the **administrator** and gains access to privileged functionality.

---

## 🧠 What is an Algorithm Confusion Attack?

An **Algorithm Confusion Attack** occurs when a JWT server supports multiple signing algorithms but **fails to strictly enforce which algorithm should be used** during verification.

For example:

- The server originally signs JWTs using **RS256**, which relies on a **private key** for signing and a **public key** for verification.
- During verification, instead of enforcing RS256, the server trusts the value supplied inside the JWT's `alg` header.
- An attacker changes the algorithm from **RS256** to **HS256**.
- The attacker then uses the server's own **public key** as the HMAC secret to sign a forged token.
- Because the server mistakenly treats the public key as a symmetric secret, the forged token is accepted.

This confusion between **asymmetric** and **symmetric** verification allows attackers to create valid administrator tokens without ever knowing the private key.

---

# Step 1: Accessing the Lab

![Screenshot 1](images/screenshot1.png)

The lab initially presents a normal web application with a **My Account** button.

At this point no authentication has been performed.

---

# Step 2: Logging into the Application

![Screenshot 2](images/screenshot2.png)

Clicking **My Account** redirects the browser to the login page.

The following valid credentials were entered:

- **Username:** `wiener`
- **Password:** `peter`

After providing the credentials, the **Log in** button was clicked.

---

# Step 3: Successful Authentication

![Screenshot 3](images/screenshot3.png)

The login succeeds and the application redirects to:

```

/my-account?id=wiener

```

This confirms that authentication is performed using JWT tokens.

At this stage the user has normal privileges.

---

# Step 4: Capturing the First JWT

![Screenshot 4](images/screenshot4.png)

The request to:

```

GET /my-account?id=wiener

```

was intercepted using Burp Suite.

Inside the request headers, the first JWT belonging to **wiener** was captured.

This token will later be compared against another JWT generated for the same user.

---

# Step 5: Capturing a Second JWT

![Screenshot 5](images/screenshot5.png)

The user logged out and authenticated again using the same credentials.

Another request to the **My Account** page was intercepted.

Although both tokens belong to the same user, they contain different signatures.

Having two valid JWTs signed by the same private key is important for the next stage of the attack.

---

# Step 6: Comparing Both Tokens

![Screenshot 6](images/screenshot6.png)

Both captured JWTs were copied into a text editor for comparison.

Although the payload remains nearly identical, the signatures differ because each JWT was independently signed by the server.

These multiple signatures provide enough information for **sig2n** to mathematically reconstruct candidate public keys.

---

# Step 7: Recovering Candidate Public Keys using sig2n

![Screenshot 7](images/screenshot7.png)

The following Docker command was executed:

```bash
docker run --rm -it portswigger/sig2n <JWT1> <JWT2>
```

### Command Breakdown

| Component | Purpose |
|-----------|----------|
| `docker run` | Starts a temporary Docker container. |
| `--rm` | Automatically removes the container after execution. |
| `-it` | Runs the container interactively. |
| `portswigger/sig2n` | Official PortSwigger image containing the **sig2n** utility. |
| `<JWT1>` | First JWT captured from the application. |
| `<JWT2>` | Second JWT captured from the same user. |

### What is sig2n?

**sig2n** is a PortSwigger utility that analyzes multiple JWT signatures created using the same RSA private key.

Using mathematical properties of RSA signatures, it generates several **candidate public keys** together with matching **tampered JWTs**.

One of these candidates corresponds to the server's real public key.

---

# Step 8: Generated Candidate Keys

![Screenshot 8](images/screenshot8.png)

The **sig2n** tool produced multiple candidate results.

Each result contains two important components:

### 📌 Base64 Encoded X.509 Key

This is a candidate RSA public key encoded using the **X.509 certificate format**.

X.509 is a standard structure used for storing and distributing public keys in certificates and cryptographic systems.

Each generated key represents one possible public key that could correspond to the server's private signing key.

### 📌 Tampered JWT

For every generated public key, **sig2n** also creates a JWT signed using that candidate key.

These JWTs are called **tampered tokens** because they have been intentionally modified for testing purposes.

The idea is to try each generated JWT against the server until one is accepted.

If a tampered JWT is accepted, its corresponding X.509 key is the correct public key needed for the algorithm confusion attack.

---

# Step 9: Additional Candidate Keys

![Screenshot 9](images/screenshot9.png)

The remaining output from **sig2n** contains several additional candidate X.509 public keys and their corresponding tampered JWTs.

Since the tool cannot determine which candidate is correct, each generated JWT must be tested individually against the target application until a valid one is found.

---

# Step 10: Testing Candidate JWTs

![Screenshot 10](images/screenshot10.png)

Each generated tampered JWT was tested by replacing the original JWT in the intercepted request.

Initially, the server responded with:

```

HTTP/1.1 302 Found
Location: /login

```

This indicates that the JWT was **not accepted**.

Instead of granting access, the application redirected the request back to the login page, meaning the tested public key was incorrect.

This trial-and-error process continues until one of the candidate JWTs is accepted.

---

# Step 11: Selecting the Correct Tampered JWT

![Screenshot 11](images/screenshot11.png)

Eventually, the **second-last tampered JWT** generated by **sig2n** was identified as the correct candidate.

This JWT was copied because it successfully corresponds to the server's real public key.

Using this candidate allows the remainder of the algorithm confusion attack to proceed successfully.

---

# Step 12: Successful Verification

![Screenshot 12](images/screenshot12.png)

After replacing the JWT with the selected tampered token, the request was sent again.

This time the server returned:

```

HTTP/1.1 200 OK

```

Receiving **200 OK** confirms that the JWT was successfully verified.

Returning to the **sig2n** output, the matching **Base64-encoded X.509 public key** corresponding to this successful JWT was copied.

This recovered public key will later be reused as the **HMAC secret** during the algorithm confusion attack.

---

# Step 13: Creating a Symmetric Key

![Screenshot 13](images/screenshot13.png)

The **JWT Editor** extension in Burp Suite was opened.

A new **Symmetric Key** was generated.

After generation, the value of the **`k`** parameter was replaced with the previously recovered **Base64-encoded X.509 public key**.

```json
{
  "kty": "oct",
  "kid": "5ad1bbb8-149a-4479-969d-591520923c0b",
  "k": "<Recovered Base64 X.509 Public Key>"
}
```

This prepares the key for the upcoming algorithm confusion attack.

Instead of using a normal shared secret, the recovered RSA public key is intentionally reused as the HMAC secret because the vulnerable server incorrectly accepts HS256 verification using the public key.
