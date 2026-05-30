# Username Enumeration via Different Responses

## 📌 Lab Overview

This lab demonstrates a **Username Enumeration** vulnerability that occurs when an application reveals different error messages during the login process.

The vulnerability occurred because:
- the application returned different responses for invalid usernames and invalid passwords
- attackers could distinguish valid usernames from invalid usernames
- valid accounts could be identified before attempting password attacks

This lab focused on:
- username enumeration
- authentication flaws
- brute-force attacks
- login response analysis
- credential discovery

---

# 🔍 What is Username Enumeration?

Username Enumeration occurs when an application reveals whether a username exists within the system.

For example:

### Invalid Username

```text
Invalid username
```

### Valid Username but Wrong Password

```text
Incorrect password
```

An attacker can use these differences to identify valid usernames without knowing the password.

A secure application should always return a generic message such as:

```text
Invalid username or password
```

---

# ⚠ Why is Username Enumeration Dangerous?

If attackers can discover valid usernames, they can:

- perform targeted brute-force attacks
- conduct password spraying attacks
- gather information about users
- increase the success rate of account compromise

---

# 🎯 Objective

The goal of this lab was to:
- identify a valid username
- discover the corresponding password
- authenticate successfully
- gain access to the target account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Different%20Responses/screenshots/lab1(1).png?raw=true)

The application initially displayed:
- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Accessing the Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Different%20Responses/screenshots/lab1(2).png?raw=true)

After clicking the **My Account** button, I landed on the login page.

I attempted to log in using random credentials:

```text
Username: abcd
Password: password
```

---

## Screenshot 3 — Discovering Username Enumeration

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Different%20Responses/screenshots/lab1(3).png?raw=true)

After submitting the credentials, the application returned:

```text
Invalid username
```

This indicated that the application was explicitly revealing whether a username existed.

This behavior suggested a potential **Username Enumeration** vulnerability.

---

## Screenshot 4 — Capturing the Login Request

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Different%20Responses/screenshots/lab1(4).png?raw=true)

I captured the login request using Burp Suite.

```http
POST /login
```

This request would later be used for automated testing.

---

## Screenshot 5 — Sending the Request to Intruder

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Username%20Enumeration%20via%20Different%20Responses/screenshots/lab1(5).png?raw=true)

The captured request was sent to Burp Intruder.

I selected two payload positions:

```text
Username: abcd
Password: password
```

These positions would be replaced automatically during the attack.

---

## Screenshot 6 — Collecting Usernames

![Screenshot 6](screenshot-auth6.png)

The lab provided a list of usernames.

I copied all usernames and prepared them for use in Burp Intruder.

---

## Screenshot 7 — Collecting Passwords

![Screenshot 7](screenshot-auth7.png)

The lab also provided a password list.

I copied all passwords and prepared them for use in Burp Intruder.

---

## Screenshot 8 — Launching the Cluster Bomb Attack

![Screenshot 8](screenshot-auth8.png)

I configured Burp Intruder using:

```text
Attack Type: Cluster Bomb
```

### What is a Cluster Bomb Attack?

A Cluster Bomb attack tests every possible combination of multiple payload sets.

Example:

```text
user1 : password1
user1 : password2
user1 : password3

user2 : password1
user2 : password2
user2 : password3
```

This allows Burp Intruder to systematically test all username and password combinations.

After running the attack, I analyzed the results and noticed a response with:

```text
Status Code: 302
```

The valid credentials were:

```text
Username: pi
Password: 1234567890
```

The 302 response indicated a successful login because the application redirected the user after authentication.

---

## Screenshot 9 — Logging In with Discovered Credentials

![Screenshot 9](screenshot-auth9.png)

I returned to the login page and entered the credentials discovered during the Intruder attack:

```text
Username: pi
Password: 1234567890
```

---

## Screenshot 10 — Successful Authentication

![Screenshot 10](screenshot-auth10.png)

The login was successful and I was authenticated as:

```text
pi
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Exists

The vulnerability occurred because the application revealed different responses during authentication.

Example:

```text
Invalid username
```

vs

```text
Incorrect password
```

This allowed attackers to determine whether a username existed before attempting password attacks.

---

# 💥 Impact

An attacker can potentially:

- identify valid usernames
- perform targeted brute-force attacks
- conduct password spraying attacks
- increase the likelihood of account compromise
- gather user information

---

# 🛡 Mitigation

To prevent Username Enumeration vulnerabilities:

- return generic authentication error messages
- use consistent response lengths
- use consistent response timing
- implement account lockout mechanisms
- enforce rate limiting
- deploy MFA where possible

Example:

```text
Invalid username or password
```

should be returned for every failed login attempt.

---

# 🧠 Skills Learned

- Username Enumeration
- Authentication Testing
- Burp Intruder
- Cluster Bomb Attacks
- Login Response Analysis
- Credential Discovery

---

# 🧰 Tools Used

- Burp Suite
- Burp Intruder
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how different authentication responses can lead to **Username Enumeration**.

By observing the application's login behavior and using Burp Intruder with a Cluster Bomb attack, I was able to identify valid credentials and successfully authenticate as a legitimate user.

Through this lab, I learned:
- how username enumeration works
- how attackers identify valid accounts
- how brute-force attacks are performed efficiently
- why authentication responses should be generic

The lab was successfully solved by discovering valid credentials through username enumeration and automated credential testing.
