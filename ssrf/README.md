# 🌐 Server-Side Request Forgery (SSRF)

This section contains my walkthroughs and notes for the **PortSwigger Web Security Academy SSRF labs**.

Through these labs, I learned how attackers can abuse server-side functionality to force applications into making unintended requests, access internal systems, bypass filters, and interact with hidden backend services.

---

## 🎯 Skills Covered

- SSRF Fundamentals
- Localhost Access
- Internal Network Enumeration
- Blind SSRF
- Out-of-Band (OAST) Detection
- Burp Collaborator
- Open Redirection Abuse
- URL Parsing Attacks
- Blacklist Bypass Techniques
- Whitelist Bypass Techniques
- URL Encoding Tricks
- Double URL Encoding
- Internal Service Discovery
- Shellshock Exploitation
- SSRF-to-RCE Chains

---

## 🧪 Labs Completed

### Basic SSRF

- ✅ Basic SSRF against the local server
- ✅ Basic SSRF against another back-end system

### Blind SSRF

- ✅ Blind SSRF with out-of-band detection
- ✅ Blind SSRF with Shellshock exploitation

### SSRF Filter Bypass

- ✅ SSRF with blacklist-based input filter
- ✅ SSRF with whitelist-based input filter
- ✅ SSRF with filter bypass via open redirection vulnerability

---

## 🛠 Tools Used

- Burp Suite
- Burp Repeater
- Burp Intruder
- Burp Collaborator
- Burp Decoder
- PortSwigger Web Security Academy

---

## 📚 Key Takeaways

These labs demonstrated how SSRF vulnerabilities can be used to:

- access localhost services
- reach internal networks
- enumerate backend systems
- bypass security filters
- exploit trust relationships between servers
- trigger out-of-band interactions
- achieve remote command execution through vulnerable internal services

A major lesson from this path was that even strong-looking defenses such as **blacklists**, **whitelists**, and **host validation** can often be bypassed through URL parsing quirks, encoding tricks, and application logic flaws.

---

## 🏆 Status

**Completed all PortSwigger SSRF Labs and documented each walkthrough in this repository.**
