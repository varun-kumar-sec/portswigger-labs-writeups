# Username Enumeration via Subtly Different Responses

## 📌 Lab Overview

This lab demonstrates a **Username Enumeration** vulnerability where the application appears to return the same error message for all failed login attempts, but subtle differences in the response reveal whether a username exists.

The vulnerability occurred because:
- the application returned slightly different responses for valid and invalid usernames
- the differences were not immediately visible to users
- attackers could identify valid usernames by carefully comparing responses

This lab focused on:
- username enumeration
- authentication flaws
- response analysis
- brute-force attacks
- Burp Intruder techniques

---

# 🔍 What is Username Enumeration?

Username Enumeration occurs when an application allows attackers to determine whether a username exists within the system.

Sometimes applications directly reveal:

```text
Invalid username
```

or

```text
Incorrect password
```

However, modern applications often attempt to hide this by returning a generic message such as:

```text
Invalid username or password.
```

Unfortunately, even when the message appears identical, subtle differences in:
- punctuation
- whitespace
- response length
- response timing

can still reveal valid usernames.

---

# ⚠ Why is This Dangerous?

If attackers discover valid usernames, they can:

- perform targeted brute-force attacks
- conduct password spraying attacks
- gather information about users
- increase the chances of account compromise

A secure application should ensure that all authentication failures produce identical responses.

---

# 🎯 Objective

The goal of this lab was to:

- identify a valid username
- discover the corresponding password
- successfully authenticate
- gain access to the target account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Subtly%20Different%20Responses/screenshots/lab4(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Accessing the Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Subtly%20Different%20Responses/screenshots/lab4(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The page contained:

```text
Username
Password
```

fields.

I attempted to log in using:

```text
Username: abcd
Password: password
```

---

## Screenshot 3 — Generic Error Message

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Subtly%20Different%20Responses/screenshots/lab4(3).png?raw=true)

After submitting the credentials, the application returned:

```text
Invalid username or password.
```

At first glance, this appeared secure because the application was not revealing whether the username or password was incorrect.

---

## Screenshot 4 — Capturing the Login Request

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Subtly%20Different%20Responses/screenshots/lab4(4).png?raw=true)

I captured the login request using Burp Suite.

```http
POST /login
```

Request parameters:

```text
username=abcd
password=password
```

This request would be used for automated testing.

---

## Screenshot 5 — Testing Usernames with Burp Intruder

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Subtly%20Different%20Responses/screenshots/lab4(5).png?raw=true)

I copied the username list provided by the lab and configured Burp Intruder.

Payload position:

```text
username=$abcd$
password=password
```

Attack type:

```text
Sniper
```

### Why Sniper?

A Sniper attack changes only one parameter at a time while keeping all other values constant.

This makes it ideal for testing usernames individually.

---

### Using Grep-Match

I configured Intruder's **Grep-Match** feature and searched for:

```text
Invalid username or password.
```

This allowed Burp Intruder to automatically detect whether the exact phrase appeared in each response.

---

## Screenshot 6 — Identifying a Suspicious Username

![Screenshot 6](screenshot-auth6.png)

After running the attack, most responses contained:

```text
1
```

in the Grep-Match column.

This indicated that Burp successfully found:

```text
Invalid username or password.
```

inside those responses.

However, one username stood out:

```text
antivirus
```

For this response:

```text
0 matches found
```

This suggested that the response differed slightly from the others.

---

## Screenshot 7 — Comparing Responses

![Screenshot 7](screenshot-auth7.png)

To identify the difference, I selected:

- one normal response
- the antivirus response

and sent both to Burp Comparer.

Using a **word-by-word comparison**, I discovered:

### Invalid Username Response

```text
Invalid username or password.
```

### Valid Username Response

```text
Invalid username or password
```

The only difference was:

```text
.
```

The full stop at the end of the message was missing.

---

# 🔍 Why This Matters

This tiny punctuation difference revealed that:

```text
antivirus
```

was a valid username.

Although visually subtle, the application's backend processed valid and invalid usernames differently.

This created a username enumeration vulnerability.

---

## Screenshot 8 — Brute-Forcing the Password

![Screenshot 8](screenshot-auth8.png)

After identifying the valid username:

```text
antivirus
```

I modified the Intruder attack:

```text
username=antivirus
password=$password$
```

I copied the password list provided by the lab and launched another Sniper attack.

After reviewing the responses, I identified the correct password:

```text
george
```

---

## Screenshot 9 — Logging In

![Screenshot 9](screenshot-auth9.png)

I returned to the login page and entered:

```text
Username: antivirus
Password: george
```

---

## Screenshot 10 — Successful Authentication

![Screenshot 10](screenshot-auth10.png)

The login was successful and I was authenticated as:

```text
antivirus
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability occurred because the application generated slightly different responses depending on whether the username existed.

Even though both responses looked almost identical:

```text
Invalid username or password.
```

and

```text
Invalid username or password
```

the difference was enough for attackers to distinguish valid usernames.

---

# 💥 Impact

An attacker can potentially:

- identify valid usernames
- perform targeted brute-force attacks
- conduct password spraying attacks
- improve account takeover success rates
- gather information about users

---

# 🛡 Mitigation

To prevent username enumeration:

- return identical error messages
- maintain identical punctuation and formatting
- keep response lengths consistent
- normalize response timing
- implement account lockouts
- enforce rate limiting
- deploy MFA where possible

Example:

```text
Invalid username or password
```

should be returned exactly the same way for every authentication failure.

---

# 🧠 Skills Learned

- Username Enumeration
- Authentication Testing
- Burp Intruder
- Grep-Match Analysis
- Burp Comparer
- Response Comparison Techniques

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- Burp Comparer
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how even tiny differences in authentication responses can lead to Username Enumeration.

By using Burp Intruder and Grep-Match, I identified a username whose response differed slightly from the others. Comparing the responses revealed a missing full stop, which exposed a valid username.

After discovering the username, I performed a password attack and successfully authenticated with the correct credentials.

Through this lab, I learned:

- how subtle response differences reveal valid usernames
- how to use Burp Intruder effectively
- how Grep-Match helps identify anomalies
- why authentication responses must be completely identical

The lab was successfully solved by exploiting username enumeration through subtle response differences and discovering valid credentials.
```**``**
