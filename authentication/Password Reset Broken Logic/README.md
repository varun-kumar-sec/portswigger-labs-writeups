# Password Reset Broken Logic

## 📌 Lab Overview

This lab demonstrates a **Password Reset Broken Logic** vulnerability.

The vulnerability occurred because:
- the application trusted user-supplied parameters during the password reset process
- the password reset token was not properly validated
- the backend relied on client-side supplied usernames instead of binding the reset request to the intended user

This lab focused on:
- password reset vulnerabilities
- authentication flaws
- token validation weaknesses
- account takeover
- business logic vulnerabilities

---

# 🔍 What is a Password Reset Vulnerability?

Most applications provide a password reset feature for users who forget their passwords.

A secure password reset workflow typically looks like:

```text
1. User requests password reset
2. Application generates a unique reset token
3. Reset token is sent to the user's email
4. User clicks the reset link
5. Application validates the token
6. Password is changed
```

The reset token acts like a temporary password.

If the application fails to properly validate this token, attackers may be able to reset passwords belonging to other users.

---

# ⚠ Why is this Dangerous?

If password reset logic is broken:

- attackers can take over accounts
- passwords can be changed without authorization
- users can be locked out of their own accounts
- sensitive information can be exposed

In many real-world breaches, password reset functionality becomes the easiest path to account compromise.

---

# 🎯 Objective

The goal of this lab was to:

- analyze the password reset workflow
- identify weaknesses in token validation
- modify the password reset request
- reset another user's password
- gain access to the target account

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(1).png?raw=true)

The application initially displayed:

- a normal webpage
- a **My Account** button

---

## Screenshot 2 — Accessing the Login Page

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(2).png?raw=true)

After clicking **My Account**, I landed on the login page.

The page contained:

- Username field
- Password field
- Forgot Password button

---

## Screenshot 3 — Initiating Password Reset

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(3).png?raw=true)

I clicked the **Forgot Password** button.

The application redirected me to a page asking:

```text
Please enter your username or email
```

I entered:

```text
wiener
```

and submitted the request.

---

## Screenshot 4 — Reset Email Notification

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(4).png?raw=true)

After submitting the username, the application displayed:

```text
Please check your email for a reset password link
```

---

## Screenshot 5 — Retrieving the Reset Link

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(5).png?raw=true)

I clicked the **Email Client** button provided by the lab.

Inside the email, I received a password reset link.

---

## Screenshot 6 — Password Reset Form

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(6).png?raw=true)

After clicking the reset link, I was redirected to a password reset page.

The page contained:

```text
New Password
Confirm New Password
```

along with a submit button.

---

## Screenshot 7 — Verifying the Reset Process

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/authentication/Password%20Reset%20Broken%20Logic/screenshots/lab3(7).png?raw=true)

I set a new password for the account and returned to the login page.

Using the updated password, I successfully logged in.

This confirmed that the password reset workflow was functioning normally.

---

## Screenshot 8 — Inspecting the Password Reset Request

![Screenshot 8](screenshot-auth8.png)

While reviewing Burp Suite's request history, I found an interesting request:

```http
POST /forgot-password?temp-forgot-password-token=59n8uwkompqwcuc9jro55s7ntvbnzw3s
```

The request body contained:

```text
temp-forgot-password-token=59n8uwkompqwcuc9jro55s7ntvbnzw3s
username=wiener
new-password-1=password
new-password-2=password
```

This revealed that the application relied on:

- a temporary reset token
- a username parameter
- the new password values

---

# 🔍 Why Was This Interesting?

In a secure implementation:

```text
Reset Token → User Account
```

should already be linked internally by the server.

The application should not trust a user-controlled username field during password reset.

---

## Screenshot 9 — Modifying the Request

![Screenshot 9](screenshot-auth9.png)

I modified the request by replacing both instances of the token:

```text
fake_token
```

I also changed:

```text
username=wiener
```

to:

```text
username=carlos
```

The modified request became:

```http
POST /forgot-password?temp-forgot-password-token=fake_token
```

Request body:

```text
temp-forgot-password-token=fake_token
username=carlos
new-password-1=password
new-password-2=password
```

---

## Screenshot 10 — Successful Password Reset

![Screenshot 10](screenshot-auth10.png)

After sending the modified request, the application responded with:

```http
302 Found
```

This indicated that the password reset request had been accepted.

The critical issue was that:

- the token was not properly validated
- the supplied username was trusted
- the password reset operation completed successfully

As a result, Carlos's password was changed.

---

## Screenshot 11 — Logging In as Carlos

![Screenshot 11](screenshot-auth11.png)

I returned to the login page and authenticated using:

```text
Username: carlos
Password: password
```

The login was successful.

The lab was solved after gaining access to Carlos's account.

---

# ⚠ Why This Vulnerability Exists

The vulnerability occurred because the application trusted user-controlled parameters during the password reset process.

Instead of verifying:

```text
Does this reset token belong to Carlos?
```

the application simply accepted:

```text
username=carlos
```

from the request.

The backend failed to properly associate the reset token with a specific account.

---

# 💥 Impact

An attacker can potentially:

- reset other users' passwords
- take over accounts
- lock legitimate users out
- bypass authentication controls
- gain unauthorized access to sensitive data

---

# 🛡 Mitigation

To prevent password reset logic flaws:

- bind reset tokens to specific users on the server
- never trust user-supplied usernames during password reset
- validate reset tokens before allowing password changes
- invalidate tokens after use
- enforce expiration times on reset tokens

Secure implementation:

```text
Reset Token → Server maps token to user
```

instead of:

```text
Reset Token + Username from Client
```

---

# 🧠 Skills Learned

- Password Reset Testing
- Authentication Logic Analysis
- Business Logic Vulnerabilities
- Account Takeover Techniques
- Burp Suite Request Manipulation
- Token Validation Testing

---

# 🧰 Tools Used

- Burp Suite
- Burp Repeater
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how broken password reset logic can lead to full account takeover.

By analyzing the password reset workflow, I discovered that the application trusted user-controlled parameters and failed to properly validate reset tokens.

After modifying the username parameter and replacing the reset token, I successfully reset another user's password and authenticated as that user.

Through this lab, I learned:

- how password reset workflows operate
- why reset tokens must be validated server-side
- how business logic flaws lead to account takeover
- why client-controlled identity parameters are dangerous

The lab was successfully solved by exploiting broken password reset logic and gaining unauthorized access to Carlos's account.
