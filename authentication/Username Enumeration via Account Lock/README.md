# Username Enumeration via Account Lock

## 📌 Lab Overview

This lab demonstrates how different application responses during account lockout can be used to enumerate valid usernames. Although the application attempts to protect accounts by locking them after multiple failed login attempts, the lockout mechanism itself leaks information about which usernames exist.

An attacker can exploit these differences to identify valid usernames before performing password attacks.

---

## What is Username Enumeration?

**Username Enumeration** occurs when an application reveals whether a username exists through different responses, status codes, timing differences, account lock messages, or other behavioral differences.

### Example

**Invalid Username**
```
Invalid username or password.
```

**Valid Username**
```
You have made too many incorrect login attempts.
Please try again in 1 minute.
```

Even though both login attempts fail, the second response confirms that the username exists because the application is tracking failed attempts for that account.

---

## Why This Vulnerability Exists

The application applies account lockout protection only to valid user accounts.

When an invalid username is supplied:

- No account exists to lock.
- The application simply returns a generic error.

When a valid username is supplied:

- Failed login attempts are tracked.
- The account eventually becomes locked.
- A different error message is displayed.

This difference allows attackers to discover legitimate usernames without knowing any passwords.

---

# 🎯 Objective

The goal of this lab was to:

1. Identify a valid username by abusing the account lock mechanism.
2. Brute-force the password for that username.
3. Log in successfully and solve the lab.

---

# Step 1 - Access the Login Page

The lab starts with a website containing a **My Account** button.

![Screenshot 1](screenshot1.png)

After clicking **My Account**, the login page appears.

I entered random credentials:

- Username: `admin`
- Password: `admin`

because the correct credentials were unknown.

![Screenshot 2](screenshot2.png)

---

# Step 2 - Observe the Error Message

After attempting to log in, the application returned:

```text
Invalid username or password.
```

This indicates the authentication attempt failed.

![Screenshot 3](screenshot3.png)

At this stage, it is impossible to determine whether:

- The username is wrong.
- The password is wrong.
- Both are wrong.

---

# Step 3 - Capture the Login Request

Using Burp Suite, I intercepted the login request.

```http
POST /login
```

The request contained:

```http
username=admin
password=admin
```

I then sent the request to **Intruder** for automated testing.

![Screenshot 4](screenshot4.png)

---

# Step 4 - Enumerate Usernames

The goal was to identify which username triggers account lockout behavior.

I configured Intruder as follows:

### Payload Position

```http
username=§admin§
password=admin
```

### Attack Type

**Cluster Bomb**

### Payload

I pasted the entire username list provided by the lab into the payload section.

This causes Burp Suite to test every username against the same password.

![Screenshot 5](screenshot5.png)

---

# Step 5 - Analyze Intruder Results

After the attack completed, I examined the responses.

A random response displayed the standard message:

```text
Invalid username or password.
```

This indicates the tested username was invalid.

![Screenshot 6](screenshot6.png)

---

# Step 6 - Identify the Valid Username

To find anomalies, I sorted Intruder results by the **Length** column.

One response stood out with a significantly larger response length:

```text
3396
```

After opening that response and viewing it in the **Render** tab, I observed a different message:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute.
```

This message only appears when the application recognizes the username and tracks failed login attempts.

Therefore, the username was confirmed to be:

```text
apache
```

![Screenshot 7](screenshot7.png)

---

## Why the Response Length Was Different

The lockout page contains:

- Additional text
- Additional HTML elements
- Different content structure

As a result:

- Invalid usernames generated shorter responses.
- Valid usernames generated longer responses.

This difference made the correct username easy to identify by sorting response lengths.

---

# Step 7 - Brute-Force the Password

After discovering the username, I modified the Intruder attack.

### Updated Request

```http
username=apache
password=§admin§
```

### Payload

I pasted the password list provided by the lab.

The attack tested each password against the valid username.

![Screenshot 8](screenshot8.png)

After reviewing the results, I found the correct password:

```text
1111
```

The successful response had the smallest response length because the application redirected the user after successful authentication.

---

# Step 8 - Log In

Using the discovered credentials:

```text
Username: apache
Password: 1111
```

I logged into the application successfully.

![Screenshot 9](screenshot9.png)

The lab was solved.

---

# Attack Flow Summary

```text
Login Page
      │
      ▼
Test Multiple Usernames
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
Brute-Force Password
      │
      ▼
Login Successfully
```

---

# Why the Attack Worked

The attack succeeded because:

1. The application produced different responses for valid and invalid usernames.
2. Account lockout functionality only applied to existing users.
3. The lockout page generated a noticeably different response length.
4. Burp Intruder made it easy to identify these differences.

---

# 💥 Security Impact

If exploited in a real application, an attacker could:

- Discover valid usernames.
- Build a list of legitimate accounts.
- Perform targeted password attacks.
- Conduct credential stuffing attacks.
- Aid phishing campaigns using known usernames.

Although no passwords are disclosed directly, username enumeration significantly increases the effectiveness of subsequent attacks.

---

# Mitigation

Organizations should prevent username enumeration by:

### Use Generic Error Messages

Always return:

```text
Invalid username or password.
```

for every authentication failure.

### Avoid Different Lockout Responses

Do not reveal whether:

- The username exists.
- The account is locked.
- The password is incorrect.

### Normalize Response Lengths

Ensure all authentication responses have similar sizes.

### Rate Limiting

Limit login attempts based on:

- IP address
- User account
- Device fingerprint

### Multi-Factor Authentication

Use MFA to reduce the impact of credential attacks.

---

# Tools Used

- Burp Suite Intruder
- Burp Suite Repeater
- Burp Suite Render View
- PortSwigger Authentication Username List
- PortSwigger Authentication Password List

---

# Skills Learned

- Username enumeration through account lockouts
- Response-length analysis
- Burp Suite Intruder configuration
- Cluster Bomb attack usage
- Authentication bypass methodology
- User discovery techniques
- Login brute-force analysis
- Response comparison techniques

---

# Conclusion

This lab demonstrated how account lockout mechanisms can unintentionally leak valid usernames. By analyzing response lengths and error messages, it was possible to identify an existing user account (`apache`) and then brute-force its password (`1111`). The vulnerability existed because the application handled valid and invalid usernames differently during authentication failures, enabling username enumeration and facilitating further attacks.
