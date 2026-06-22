# JWT (JSON Web Token) - Complete Introduction

## 📌 What You Will Learn

In this section, we will cover:

1. What is JSON?
2. What is a Token?
3. What is JWT?
4. JWT Format
5. Types of JWT
6. JWT Attacks & Impact
7. What is a Signature and How is it Created?

---

# 01) What is JSON?

## Definition

**JSON (JavaScript Object Notation)** is a lightweight data-interchange format used to store and transfer structured information between systems.

It organizes data in the form of:

```text
Key : Value
```

pairs.

---

## Example

```json
{
    "username":"wiener",
    "role":"user",
    "admin":false
}
```

---

## Breaking Down the Example

```json
{
    "username":"wiener",
    "role":"user",
    "admin":false
}
```

### username

```json
"username":"wiener"
```

Meaning:

```text
Key   → username
Value → wiener
```

Stores the username of the user.

---

### role

```json
"role":"user"
```

Meaning:

```text
Key   → role
Value → user
```

Stores the user's role.

---

### admin

```json
"admin":false
```

Meaning:

```text
Key   → admin
Value → false
```

Indicates that the user does not have administrative privileges.

---

## Why JSON Exists

Without JSON:

```text
username=wiener
role=user
admin=false
```

Data becomes harder to organize and parse.

JSON provides:

```text
Structured Data
Human Readable Format
Machine Readable Format
Easy Parsing
```

which is why APIs heavily use JSON.

---

# 02) What is a Token?

## Definition

A token is a piece of data used to prove identity or authorization.

Think of it as a digital pass.

---

## Real-World Example

Imagine purchasing a movie ticket.

You receive:

```text
Ticket No: A123
```

When entering the theater:

```text
Security checks ticket
↓
Ticket Valid
↓
Access Granted
```

The ticket proves:

```text
You paid
You are authorized to enter
```

---

## Web Application Equivalent

User logs in:

```text
Username: wiener
Password: peter
```

Server verifies credentials.

Instead of asking for the password every time:

```text
Server Issues Token
```

Example:

```text
abc123xyz
```

Browser stores the token.

Future requests contain:

```http
Authorization: Bearer abc123xyz
```

The token proves:

```text
User Already Authenticated
```

---

## Why Tokens Exist

Without tokens:

```text
Login
↓
Request
↓
Login Again
↓
Request
↓
Login Again
```

With tokens:

```text
Login Once
↓
Receive Token
↓
Reuse Token
```

This improves:

```text
Performance
User Experience
Scalability
```

---

# 03) What is JWT?

## Definition

JWT stands for:

```text
JSON Web Token
```

It is a token format that stores information using JSON and protects it using a cryptographic signature.

---

## Why JWT Was Created

### Traditional Authentication

```text
Login
↓
Server Creates Session
↓
Server Stores Session
↓
User Receives Session ID
```

Server must maintain session data.

---

### JWT Authentication

```text
Login
↓
Server Creates JWT
↓
Browser Stores JWT
↓
JWT Sent With Requests
```

The server does not need to store session information.

This is called:

```text
Stateless Authentication
```

---

## Example JWT Payload

```json
{
    "username":"wiener",
    "role":"user"
}
```

---

## Breaking Down the Example

### username

```json
"username":"wiener"
```

Identifies the authenticated user.

---

### role

```json
"role":"user"
```

Determines what permissions the user has.

The server may later check:

```text
role = admin ?
```

before allowing access to administrative functionality.

---

# 04) JWT Format

A JWT consists of three parts:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VybmFtZSI6IndpZW5lciIsImFkbWluIjpmYWxzZX0
.
abcdef123456
```

---

## Part 1 — Header

### Example

```json
{
    "alg":"HS256",
    "typ":"JWT"
}
```

---

### Breaking Down the Header

#### alg

```json
"alg":"HS256"
```

Meaning:

```text
Algorithm = HS256
```

Tells the server:

```text
Use HMAC SHA256
```

to verify the signature.

---

#### typ

```json
"typ":"JWT"
```

Meaning:

```text
Token Type = JWT
```

Used to identify the token format.

---

## Part 2 — Payload

### Example

```json
{
    "username":"wiener",
    "admin":false
}
```

---

### Breaking Down the Payload

#### username

```json
"username":"wiener"
```

Stores user identity.

---

#### admin

```json
"admin":false
```

Stores privilege information.

Application may later evaluate:

```text
admin == true
```

before granting administrative access.

---

## Part 3 — Signature

Example:

```text
abcdef123456
```

---

## Why Signature Exists

Suppose attacker changes:

```json
{
    "admin":false
}
```

to:

```json
{
    "admin":true
}
```

Without a signature:

```text
Server Cannot Detect Tampering
```

With a signature:

```text
Modified Payload
↓
Signature Invalid
↓
Access Denied
```

---

## JWT Structure Visualization

```text
┌──────────┐
│ Header   │
└──────────┘
      .
┌──────────┐
│ Payload  │
└──────────┘
      .
┌──────────┐
│ Signature│
└──────────┘
```

---

# 05) Types of JWT

## 1. Signed JWT (JWS)

Most common JWT type.

Structure:

```text
Header
Payload
Signature
```

Purpose:

```text
Integrity Protection
```

---

### Example

```json
{
    "username":"wiener",
    "role":"user"
}
```

Anyone can read it.

However:

```text
Nobody Can Modify It
```

without generating a valid signature.

---

## 2. Encrypted JWT (JWE)

Purpose:

```text
Confidentiality
```

Entire payload is encrypted.

---

### Difference

#### JWS

```text
Readable
Signed
```

#### JWE

```text
Encrypted
Protected
```

---

# 06) JWT Attacks & Impact

## 1. Information Disclosure

### Example

```json
{
    "username":"wiener",
    "role":"admin"
}
```

Anyone can decode this.

---

### Why?

JWT uses:

```text
Base64 Encoding
```

not encryption.

Many developers mistakenly believe:

```text
Encoded = Secure
```

which is incorrect.

---

## 2. Weak Secret Attack

### Example

Secret Key:

```text
password123
```

Attacker brute-forces the secret.

---

### What Happens Next?

Original:

```json
{
    "admin":false
}
```

Attacker modifies:

```json
{
    "admin":true
}
```

Creates valid signature.

Server accepts token.

Result:

```text
Privilege Escalation
```

---

## 3. alg:none Attack

Header:

```json
{
    "alg":"none"
}
```

---

### What Happens?

Some vulnerable applications trust:

```text
alg = none
```

and skip signature verification.

Attacker creates:

```json
{
    "admin":true
}
```

without a signature.

Result:

```text
Authentication Bypass
```

---

## 4. JWK Injection

Header:

```json
{
    "jwk": {...}
}
```

---

### What Happens?

Attacker inserts their own public key.

Server trusts attacker's key.

Attacker signs arbitrary tokens.

Result:

```text
Admin Access
```

---

## 5. JKU Injection

Header:

```json
{
    "jku":"https://attacker.com/jwks.json"
}
```

---

### What Happens?

Server downloads attacker's key.

Uses it to verify attacker-created tokens.

Result:

```text
Authentication Bypass
```

---

## 6. kid Injection

Header:

```json
{
    "kid":"../../../../dev/null"
}
```

---

### What Happens?

Server loads unintended files as signing keys.

Can lead to:

```text
Key Confusion
Authentication Bypass
File Access
```

---

# Impact of JWT Vulnerabilities

Successful exploitation can lead to:

```text
Authentication Bypass
Privilege Escalation
Admin Access
Account Takeover
Sensitive Data Exposure
API Abuse
Complete Application Compromise
```

---

# 07) What is a Signature?

## Definition

A signature is a cryptographic value used to verify:

```text
Who Created The Token
Whether The Token Was Modified
```

---

## Real World Example

Think of a royal seal.

```text
Letter
+
Royal Seal
```

If someone modifies the letter:

```text
Seal Breaks
```

and tampering becomes obvious.

JWT signatures work similarly.

---

# How is a Signature Created?

Assume:

---

## Header

```json
{
    "alg":"HS256"
}
```

---

## Payload

```json
{
    "username":"wiener"
}
```

---

## Secret

```text
mysecretkey
```

---

## Server Process

### Step 1

```text
Base64(Header)
```

---

### Step 2

```text
Base64(Payload)
```

---

### Step 3

```text
Header.Payload
```

becomes:

```text
xxxxx.yyyyy
```

---

### Step 4

Apply:

```text
HMAC SHA256
```

using:

```text
mysecretkey
```

---

## Result

```text
Signature
```

Generated output:

```text
abcdef123456
```

---

## Final JWT

```text
HEADER.PAYLOAD.SIGNATURE
```

Example:

```text
xxxxx.yyyyy.abcdef123456
```

---

## JWT Signature Flow

```text
Header
   +
Payload
   +
Secret Key
        ↓
   HMAC SHA256
        ↓
    Signature
```

---

# 🧠 Key Takeaway

JWT is usually:

```text
Encoded
+
Signed
```

It is **NOT encrypted by default**.

Anyone can usually read:

```text
Header
Payload
```

but only someone with the correct signing key should be able to create a valid:

```text
Signature
```

Understanding this distinction is the foundation of JWT security and almost every JWT vulnerability encountered during JWT penetration testing and PortSwigger JWT labs.
