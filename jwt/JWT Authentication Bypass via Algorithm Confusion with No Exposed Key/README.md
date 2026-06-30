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

---

# Step 14: Requesting the Administrator Page Again

![Screenshot 14](images/screenshot14.png)

After preparing the symmetric key using the recovered **Base64-encoded X.509 public key**, I navigated back to the application and once again attempted to access the administrator panel by modifying the URL to:

```text
https://0a6800da046b638b83409b9800b200c8.web-security-academy.net/admin
```

This request would later be intercepted so that the JWT could be modified and re-signed using the newly created symmetric key.

---

# Step 15: Capturing the Administrator Request

![Screenshot 15](images/screenshot15.png)

The request to the administrator endpoint was captured in Burp Suite Repeater.

```http
GET /admin
```

Since the request still contained the original JWT, sending it resulted in:

```http
HTTP/1.1 401 Unauthorized
```

This confirms that the application still recognized the current user as **wiener**, who does not have administrator privileges.

The next step was to modify and re-sign the JWT.

---

# Step 16: Performing the Algorithm Confusion Attack

![Screenshot 16](images/screenshot16.png)

I opened the **JSON Web Token** editor available in Burp Suite's request banner.

Burp automatically decoded both the JWT header and payload.

### Header Modification

The algorithm was changed from:

```json
{
    "alg": "RS256"
}
```

to

```json
{
    "alg": "HS256"
}
```

### Payload Modification

The subject claim was changed from:

```json
"sub": "wiener"
```

to

```json
"sub": "administrator"
```

Finally, I clicked the **Sign** button and selected the symmetric key created earlier, whose secret was the recovered **Base64-encoded X.509 public key**.

Burp generated a brand-new **HS256 signature** for the modified JWT.

### 🔍 Why did this work?

Normally:

- **RS256**
  - Private key → Signing
  - Public key → Verification

However, the vulnerable application trusted the **`alg`** value supplied by the client.

By changing the algorithm to **HS256**, the server mistakenly switched to HMAC verification.

Instead of using a real shared secret, it incorrectly used its own RSA public key as the HMAC secret.

Since we had already recovered that public key using **sig2n**, we could generate a perfectly valid HS256 signature that the vulnerable server accepted.

---

# Step 17: Successfully Accessing the Administrator Panel

![Screenshot 17](images/screenshot17.png)

After sending the newly signed JWT, the application returned:

```http
HTTP/1.1 200 OK
```

This indicates that the forged JWT was accepted successfully.

Because the JWT now contained:

```json
"sub": "administrator"
```

the application treated the request as originating from the administrator account.

Scrolling through the response revealed an administrative function for deleting users:

```text
/admin/delete?username=carlos
```

Since the lab specifically requires deleting Carlos' account, I copied this endpoint for the next step.

---

# Step 18: Deleting Carlos

![Screenshot 18](images/screenshot18.png)

The intercepted request was modified to:

```http
GET /admin/delete?username=carlos
```

After sending the request, the server responded with:

```http
HTTP/1.1 302 Found
```

A **302 Found** response indicates that the deletion request was processed successfully and the application redirected the browser to another page.

I clicked **Follow Redirection** to continue.

---

# Step 19: Following the Redirect

![Screenshot 19](images/screenshot19.png)

After following the redirect, Burp Suite displayed:

```http
HTTP/1.1 200 OK
```

This confirms that the deletion request completed successfully.

---

# Step 20: Viewing the Response in Browser

![Screenshot 20](images/screenshot20.png)

To verify the response visually, I used Burp Suite's **Show Response in Browser** feature.

Burp generated a temporary URL representing the HTTP response.

I copied the generated URL.

---

# Step 21: Lab Successfully Solved

![Screenshot 21](images/screenshot21.png)

The copied URL was pasted into the browser.

After loading the page, the application displayed the message:

```text
User deleted successfully
```

This confirms that:

- The forged JWT was accepted.
- Administrator privileges were obtained.
- Carlos' account was deleted successfully.

As a result, the lab was solved successfully.

---

# 💡 Why This Attack Worked

Unlike the previous algorithm confusion lab, this application **did not expose its public key** through an endpoint such as:

```text
/jwks.json
```

However, multiple JWTs signed using the same RSA private key leaked enough mathematical information for **PortSwigger's sig2n** tool to reconstruct several candidate public keys.

The attack worked as follows:

1. Two valid JWTs belonging to the same user were collected.
2. **sig2n** analyzed both signatures and generated multiple candidate X.509 public keys.
3. Each generated tampered JWT was tested until one was accepted.
4. The corresponding Base64-encoded X.509 public key was recovered.
5. That recovered public key was used as the HMAC secret.
6. The JWT algorithm was changed from **RS256** to **HS256**.
7. The `sub` claim was modified to `administrator`.
8. The JWT was re-signed using the recovered public key.
9. Because the server incorrectly trusted the algorithm supplied by the client, it accepted the forged token.

The vulnerability was **not** the exposure of the public key.

The vulnerability was the application's failure to enforce the expected signing algorithm.

---

# 🛡️ Mitigation

To prevent Algorithm Confusion attacks:

- Never trust the **`alg`** value supplied inside the JWT.
- Explicitly enforce the expected signing algorithm on the server.
- Never allow automatic switching between **RS256** and **HS256**.
- Use separate verification logic for symmetric and asymmetric algorithms.
- Validate both the key type and algorithm before verifying signatures.
- Rotate cryptographic keys regularly.
- Keep JWT libraries updated to versions that prevent algorithm confusion attacks.
- Avoid exposing unnecessary cryptographic information wherever possible.

---

# 📚 Learning

From this lab, I learned:

- How Algorithm Confusion attacks work when the public key is **not publicly exposed**.
- How to collect multiple JWTs for the same user.
- How **PortSwigger's sig2n** tool reconstructs candidate RSA public keys.
- What an **X.509 public key** is.
- Why Base64-encoded X.509 keys are used during this attack.
- How to identify the correct candidate public key by testing tampered JWTs.
- How to create a symmetric key using the recovered public key.
- How to modify JWT claims.
- How to re-sign JWTs using Burp Suite.
- How incorrect JWT verification can result in complete privilege escalation.

---

# ✅ Conclusion

In this lab, the server did not expose its public key directly. Instead, multiple JWTs signed with the same RSA private key were analyzed using **sig2n**, allowing the recovery of candidate public keys.

After identifying the correct public key, I performed an **Algorithm Confusion attack** by changing the JWT algorithm from **RS256** to **HS256**, modifying the `sub` claim to **administrator**, and signing the token using the recovered public key as the HMAC secret.

Because the application incorrectly trusted the client-supplied algorithm, it accepted the forged JWT, granted administrator privileges, and allowed the deletion of Carlos' account, successfully solving the lab.
