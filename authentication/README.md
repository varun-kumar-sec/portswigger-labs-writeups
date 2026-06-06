# 🔐 Authentication Vulnerabilities

This section contains my walkthroughs and notes for PortSwigger Web Security Academy's **Authentication Vulnerabilities** labs.

These labs focus on flaws in login mechanisms, password reset functionality, session management, brute force protection, and multi-factor authentication (2FA).

---

## 📚 Labs Covered

### Username Enumeration

- Username enumeration via different responses
- Username enumeration via subtly different responses
- Username enumeration via response timing
- Username enumeration via account lock

### Brute Force Attacks

- Broken brute force protection, IP block
- Broken brute-force protection, multiple credentials per request
- Password brute-force via password change

### Two-Factor Authentication

- 2FA simple bypass
- 2FA bypass using a brute-force attack

### Password Reset Vulnerabilities

- Password reset broken logic
- Password reset poisoning via middleware

### Session & Cookie Attacks

- Brute-forcing a stay-logged-in cookie
- Offline password cracking

---

## 🧠 Skills Learned

- Authentication Testing
- Username Enumeration
- Brute Force Attacks
- 2FA Bypass Techniques
- Password Reset Exploitation
- Session Management Testing
- Cookie Analysis & Cracking
- Burp Suite Automation

---

## 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- Burp Decoder
- Burp Macros
- Session Handling Rules
- CrackStation

---

## 🎯 Summary

Through these labs, I learned how attackers identify valid usernames, bypass authentication controls, abuse password reset workflows, crack authentication cookies, and exploit weaknesses in 2FA implementations to achieve account takeover.
