# Username Enumeration via Account Lock

## 📌 Lab Overview

This lab demonstrates a **Username Enumeration** vulnerability through an improperly implemented **account lockout mechanism**.

The application attempted to protect user accounts from brute-force attacks by temporarily locking accounts after multiple failed login attempts. However, the lockout functionality behaved differently for valid and invalid usernames.

This difference allowed an attacker to:

- identify valid usernames
- enumerate accounts without knowing passwords
- leverage account lock messages as an information disclosure channel
- perform targeted password attacks

This lab focused on:

- authentication weaknesses
- username enumeration
- account lock analysis
- Burp Intruder automation
- response comparison techniques

---

# 🔍 What is Username Enumeration?

**Username Enumeration** occurs when an application unintentionally reveals whether a username exists within the system.

Attackers can identify valid usernames through:

- error messages
- response timing
- status codes
- account lock notifications
- password reset functionality

### Example

For an invalid username:

```text
Invalid username or password.
```

For a valid username:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute.
```

Although both requests fail, the second message confirms that the username exists because the application is tracking failed attempts for that account.

---

# ⚠ Why Does This Vulnerability Exist?

The vulnerability exists because the application handles valid and invalid usernames differently.

For invalid usernames:

```text
Invalid username or password.
```

For valid usernames after multiple failures:

```text
Account locked.
```

This difference leaks information about which usernames exist in the application.

A secure authentication system should never reveal whether:

- a username exists
- a password is incorrect
- an account is locked

---

# 🎯 Objective

The goal of this lab was to:

- identify a valid username
- exploit the account lock mechanism
- enumerate users
- brute-force the password
- gain access to the target account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Account%20Lock/screenshots/lab7(1).png?raw=true)

The application initially displayed a normal webpage containing a:

- **My Account** button

---

## Screenshot 2 — Accessing the Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Account%20Lock/screenshots/lab7(2).png?raw=true)

After clicking **My Account**, I was redirected to the login page.

Since the credentials were unknown, I entered:

```text
Username: admin
Password: admin
```

to observe the application's behavior.

---

## Screenshot 3 — Authentication Failure

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Account%20Lock/screenshots/lab7(3).png?raw=true)

After submitting the credentials, the application returned:

```text
Invalid username or password.
```

At this point, it was impossible to determine whether:

- the username was invalid
- the password was invalid
- both were incorrect

---

## Screenshot 4 — Capturing the Login Request

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Account%20Lock/screenshots/lab7(4).png?raw=true)

Using Burp Suite, I intercepted the login request:

```http
POST /login
```

The request contained:

```http
username=admin
password=admin
```

I then sent the request to **Burp Intruder** for automated testing.

---

## Screenshot 5 — Configuring Username Enumeration

![Screenshot 5](screenshot5.png)

Inside Intruder, I selected the username parameter:

```http
username=§admin§
password=admin
```

Attack Type:

```text
Cluster Bomb
```

I then pasted the username list provided by the lab into the payload section.

The goal was to determine whether any username generated a different response.

---

# 🔍 Understanding Cluster Bomb

A **Cluster Bomb** attack tests every possible combination between payload sets.

Example:

```text
Payload Set 1:
alice
bob

Payload Set 2:
password
123456
```

Generated Requests:

```text
alice : password
alice : 123456
bob : password
bob : 123456
```

In this lab, Cluster Bomb allowed automated testing of usernames against a fixed password.

---

## Screenshot 6 — Analyzing Intruder Results

![Screenshot 6](screenshot6.png)

After the attack completed, I examined a random response.

The rendered response still displayed:

```text
Invalid username or password.
```

indicating that the tested username was likely invalid.

---

## Screenshot 7 — Identifying the Valid Username

![Screenshot 7](screenshot7.png)

To locate anomalies, I sorted Intruder results by the **Length** column.

One response stood out with a significantly larger length:

```text
3396
```

After opening the response and selecting the **Render** tab, I observed a different error message:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute.
```

This message only appears when the application recognizes the username and tracks failed login attempts.

Therefore, the username was confirmed as:

```text
apache
```

---

# 🔍 Why Was Response Length Important?

Different application responses generate different page sizes.

Example:

### Invalid Username

```text
Invalid username or password.
```

Shorter response.

### Valid Username

```text
You have made too many incorrect login attempts.
```

Longer response.

Because the account lock page contains more content, the response length increases.

This made it easy to identify the valid username by sorting Intruder results.

---

## Screenshot 8 — Brute Forcing the Password

![Screenshot 8](screenshot8.png)

Once the username was identified, I modified the Intruder request:

```http
username=apache
password=§admin§
```

I then pasted the password list provided by the lab and launched the attack.

After reviewing the results, I discovered the password:

```text
1111
```

The successful response had a noticeably different response length.

---

## Screenshot 9 — Logging In

![Screenshot 9](screenshot9.png)

Using the discovered credentials:

```text
Username: apache
Password: 1111
```

I successfully authenticated to the application.

The lab was solved.

---

# 🔄 Attack Flow

```text
Access Login Page
        │
        ▼
Capture Login Request
        │
        ▼
Enumerate Usernames
        │
        ▼
Trigger Account Lock
        │
        ▼
Different Error Message Appears
        │
        ▼
Identify Valid Username
        │
        ▼
Brute Force Password
        │
        ▼
Successful Login
```

---

# ⚠ Why the Attack Worked

The attack succeeded because:

- valid usernames triggered account lock tracking
- invalid usernames did not
- different error messages were displayed
- response lengths varied significantly
- account lock status leaked sensitive information

The lockout mechanism unintentionally became a username disclosure channel.

---

# 💥 Impact

In a real-world application, an attacker could:

- enumerate valid usernames
- build lists of legitimate accounts
- perform targeted password attacks
- launch credential stuffing attacks
- assist phishing campaigns
- increase the success rate of brute-force attacks

Although no password is disclosed directly, username enumeration greatly improves an attacker's chances of compromise.

---

# 🛡 Mitigation

To prevent username enumeration:

### Use Generic Responses

Always return:

```text
Invalid username or password.
```

for every authentication failure.

### Hide Account Lock Information

Do not reveal:

- account existence
- account lock status
- password validity

### Normalize Response Lengths

Ensure all authentication failures generate similar-sized responses.

### Implement Rate Limiting

Restrict excessive authentication attempts.

### Enable Multi-Factor Authentication

Reduce the impact of credential attacks.

### Monitor Authentication Events

Detect and block enumeration attempts.

---

# 🧠 Skills Learned

- Username Enumeration
- Authentication Testing
- Account Lock Analysis
- Burp Intruder
- Cluster Bomb Attacks
- Response Length Analysis
- Login Workflow Analysis
- User Discovery Techniques

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Repeater
- Burp Comparer
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how an account lockout mechanism can unintentionally leak valid usernames through different responses and response lengths.

By analyzing authentication behavior, identifying the account lock message, and leveraging Burp Intruder to automate testing, I was able to enumerate a valid user account (`apache`) and subsequently discover its password (`1111`).

Through this lab, I learned:

- how username enumeration works
- how account lockouts can leak sensitive information
- how response length analysis can reveal valid accounts
- how Burp Intruder can automate authentication testing

The lab was successfully solved by exploiting information disclosure within the account lockout mechanism and using the discovered username to perform password enumeration.
