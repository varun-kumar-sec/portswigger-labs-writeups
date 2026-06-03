# Username Enumeration via Account Lock

**Lab Name:** Username Enumeration via Account Lock  
**Difficulty:** Practitioner

## Description

This lab demonstrates how account lockout mechanisms can unintentionally leak valid usernames. The application locks accounts after multiple failed login attempts, but the response message changes when a valid username is targeted. By observing these differences, an attacker can enumerate valid usernames and then brute-force the password.

---

## What is Username Enumeration?

Username enumeration occurs when an application reveals whether a username exists in the system. This can happen through:

- Different error messages
- Different response lengths
- Different response times
- Account lockout behavior

Once an attacker discovers a valid username, password attacks become much easier because they only need to guess the password for an existing account.

---

## Step 1: Accessing the Login Page

The lab starts with a normal web application containing a **My Account** button.

[image !][screenshot 1.png]

After clicking **My Account**, I was redirected to the login page. Since the credentials were unknown, I entered:

- Username: `admin`
- Password: `admin`

[image !][screenshot 2.png]

---

## Step 2: Observing the Login Error

After submitting the credentials, the application returned the following error message:

```text
Invalid username or password.
```

This indicates that the login attempt failed.

[image !][screenshot 3.png]

---

## Step 3: Capturing the Login Request

I captured the login request using Burp Suite.

The request was:

```http
POST /login
```

It contained the following parameters:

```http
username=admin
password=admin
```

To enumerate usernames, I sent the request to **Intruder**.

[image !][screenshot 4.png]

---

## Step 4: Brute-Forcing Usernames

Inside Intruder, I selected the username parameter and configured the attack as:

```http
username=$admin$
password=admin
```

### Why Cluster Bomb?

Although only the username payload was required here, Cluster Bomb allows testing combinations of payload positions. The goal was to test every username from the provided wordlist while keeping the password constant.

I pasted the username list supplied by the lab into the payload section and started the attack.

[image !][screenshot 4.png]

---

## Step 5: Analyzing Normal Responses

After the attack completed, I selected a random response and viewed it using the **Render** tab.

The application displayed:

```text
Invalid username or password.
```

This was the normal response for invalid usernames.

[image !][screenshot 5.png]

---

## Step 6: Finding the Valid Username

Next, I sorted the Intruder results by the **Length** column in descending order.

One response stood out because it had a significantly larger length:

```text
3396
```

After opening that response and viewing it in the Render tab, I noticed a different message:

```text
You have made too many incorrect login attempts.
Please try again in 1 minute.
```

[image !][screenshot 6.png]

### Why Does This Reveal the Username?

The application only locks accounts after multiple failed login attempts against a **valid account**.

For invalid usernames:

```text
Invalid username or password.
```

For valid usernames:

```text
You have made too many incorrect login attempts.
```

This difference confirms that the username exists.

The valid username discovered was:

```text
apache
```

---

## Step 7: Brute-Forcing the Password

Now that the username was known, I modified the Intruder attack to target the password parameter:

```http
username=apache
password=$admin$
```

I pasted the password list provided by the lab into the payload section and started the attack.

[image !][screenshot 7.png]

---

## Step 8: Identifying the Correct Password

After the attack completed, I sorted the results by response length.

The successful login response had the smallest response length because successful authentication generated a different page structure and redirect behavior.

The correct password discovered was:

```text
1111
```

[image !][screenshot 7.png]

---

## Step 9: Logging In

Using the discovered credentials:

```text
Username: apache
Password: 1111
```

I returned to the login page and authenticated successfully.

[image !][screenshot 8.png]

---

## Step 10: Lab Solved

After logging in with the valid credentials, I was successfully authenticated as the user:

```text
apache
```

The lab was solved.

[image !][screenshot 9.png]

---

# Key Takeaways

- Account lockout mechanisms can unintentionally reveal valid usernames.
- Different responses are often enough to perform username enumeration.
- Response length is a useful indicator when messages are not immediately obvious.
- Once a valid username is discovered, password brute-forcing becomes significantly easier.
- Authentication systems should return identical responses regardless of whether the username exists.

---

# Conclusion

In this lab, I exploited an information disclosure issue in the account lockout mechanism. By sending multiple login attempts and comparing response lengths, I identified a valid username because the application displayed a different lockout message for existing accounts. After discovering the username `apache`, I performed a password attack and found the correct password `1111`. Using these credentials, I successfully logged in and solved the lab.
