# 🔐 JWT (JSON Web Token)

This directory contains my complete learning journey and hands-on lab solutions for **JWT (JSON Web Token)** vulnerabilities from **PortSwigger Web Security Academy**.

JWT is one of the most widely used authentication mechanisms in modern web applications. While JWT simplifies stateless authentication, improper implementation can introduce critical security vulnerabilities such as authentication bypass, privilege escalation, and complete account compromise.

In this module, I studied JWT fundamentals, understood how tokens are created and verified, and exploited multiple real-world JWT implementation flaws through PortSwigger labs.

---

# 📚 Topics Covered

- What is JSON?
- What is a Token?
- What is JWT?
- JWT Structure (Header, Payload, Signature)
- Types of JWT
- JWT Signatures
- JWT Algorithms (HS256 & RS256)
- Common JWT Vulnerabilities
- JWT Security Best Practices

---

# 🧪 Labs Solved

| No. | Lab Name | Status |
|:---:|----------|:------:|
| 1 | JWT authentication bypass via unverified signature | ✅ |
| 2 | JWT authentication bypass via flawed signature verification | ✅ |
| 3 | JWT authentication bypass via weak signing key | ✅ |
| 4 | JWT authentication bypass via JWK header injection | ✅ |
| 5 | JWT authentication bypass via JKU header injection | ✅ |
| 6 | JWT authentication bypass via `kid` header path traversal | ✅ |
| 7 | JWT authentication bypass via algorithm confusion | ✅ |
| 8 | JWT authentication bypass via algorithm confusion with no exposed key | ✅ |

---

# 🎯 Skills Learned

Throughout these labs, I gained practical experience in:

- Understanding JWT internals
- Decoding and analyzing JWTs
- Manipulating JWT claims
- JWT signature verification
- Exploiting the `alg: none` vulnerability
- Cracking weak HMAC secrets using Hashcat
- Forging JWT signatures
- JWK Header Injection
- JKU Header Injection
- `kid` Header Path Traversal
- Algorithm Confusion attacks
- Recovering RSA public keys using **sig2n**
- Using Burp Suite JWT Editor effectively
- Privilege escalation through JWT manipulation
- Identifying and mitigating insecure JWT implementations

---

# 🛠️ Tools Used

- Burp Suite Professional
- Burp Suite JWT Editor Extension
- Burp Decoder
- Hashcat
- Docker
- PortSwigger **sig2n**
- Kali Linux
- PortSwigger Web Security Academy

---

# 📖 Documentation Style

Each lab write-up includes:

- 📌 Lab Objective
- 📖 Concept Explanation
- 🧠 Attack Background
- 📸 Step-by-step screenshots
- 🔍 Detailed explanation of every step
- 💡 Why the attack worked
- 🛡️ Mitigation
- 📚 Key Learning
- ✅ Conclusion

---

# 🎓 Learning Outcome

After completing this module, I developed a strong practical understanding of JWT authentication and learned how insecure implementations can lead to authentication bypass and privilege escalation. I also gained hands-on experience with several advanced JWT attacks and the tools commonly used to identify and exploit them during penetration testing.

---

> **Platform:** PortSwigger Web Security Academy  
> **Category:** JWT (JSON Web Token)  
> **Labs Completed:** **8 / 8** ✅
