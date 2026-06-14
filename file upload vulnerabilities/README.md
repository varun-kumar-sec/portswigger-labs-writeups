# 📂 File Upload Vulnerabilities

## Overview

This section covers File Upload Vulnerabilities from PortSwigger Web Security Academy. These labs demonstrate how insecure file upload functionality can lead to Remote Code Execution (RCE), arbitrary file uploads, path traversal, blacklist bypasses, race conditions, and server compromise.

### Skills Practiced

- File Upload Testing
- Web Shell Uploads
- Content-Type Validation Bypass
- Path Traversal
- Blacklist & Whitelist Bypass
- Null Byte Injection
- Polyglot Files
- Race Condition Exploitation
- Remote Code Execution (RCE)

---

## Labs Completed

### 1. Remote Code Execution via Web Shell Upload
- Uploaded a PHP web shell directly.
- Executed server-side PHP code.
- Retrieved sensitive data from the server.

### 2. Web Shell Upload via Content-Type Restriction Bypass
- Bypassed Content-Type validation.
- Uploaded a PHP web shell disguised as an image.
- Achieved Remote Code Execution.

### 3. Web Shell Upload via Path Traversal
- Exploited path traversal in filename handling.
- Uploaded a PHP file outside the intended upload directory.
- Executed the uploaded web shell.

### 4. Web Shell via Extension Blacklist Bypass
- Bypassed extension filtering using Apache `.htaccess`.
- Introduced a custom executable extension.
- Executed malicious PHP code.

### 5. Web Shell Upload via Obfuscated File Extension
- Used null byte injection (`%00`) to bypass extension validation.
- Uploaded an executable PHP file while satisfying upload restrictions.
- Retrieved sensitive server data.

### 6. Remote Code Execution via Polyglot Web Shell Upload
- Created a polyglot file that was both a valid image and valid PHP.
- Embedded PHP payload inside image metadata.
- Bypassed image validation and achieved code execution.

### 7. Web Shell Upload via Race Condition
- Exploited a Time-of-Check to Time-of-Use (TOCTOU) vulnerability.
- Accessed the uploaded PHP file before it was deleted.
- Retrieved the target secret through race condition exploitation.

---

## Key Takeaways

Through these labs, I learned how attackers can abuse insecure file upload functionality to:

- Upload arbitrary files
- Bypass validation controls
- Execute server-side code
- Read sensitive files
- Gain Remote Code Execution (RCE)
- Exploit race conditions and path traversal flaws

I also gained hands-on experience with:

- Burp Suite
- Repeater
- Intruder
- Parallel Request Attacks
- EXIF Metadata Manipulation
- Apache Configuration Abuse
- File Signature Validation Testing

---

## Status

✅ All File Upload Vulnerability Labs Completed
