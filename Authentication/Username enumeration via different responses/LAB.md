# Username enumeration via different responses

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)  
**Category:** Authentication vulnerabilities  
**Difficulty:** Apprentice

## Objective
To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

## Vulnerability
This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

[Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)  
[Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Exploitation

### Steps

1. Attempted to log in using `test:test` and intercepted the HTTP request.

2. Sent the request to Burp Intruder and configured a **Sniper** attack on the `username` parameter. Added usernames from the candidate usernames wordlist and started the attack.

3. Identified the valid username by comparing responses:
    - Invalid usernames returned the message `Invalid username`.
    - The username `alerts` returned `Incorrect password` and had a different response length, indicating that the account exists.

4. Sent the request again to Burp Intruder, replaced the username value with `alerts`, and configured a **Sniper** attack on the `password` parameter. Added passwords from the candidate passwords wordlist.

5. Identified the correct password by comparing responses:
    - Most attempts returned HTTP status code `200`.
    - The password `12345` returned HTTP status code `302` and a different response length, indicating a successful login.

6. Logged in through the browser using the credentials `alerts:12345` and successfully accessed the account page.

### Payload

Username enumeration:

```http
username=§test§&password=test
```

Password brute-force:

```http
username=alerts&password=§test§
```

### Shortcut

1. Send the login request to Burp Intruder.

2. Enumerate usernames using a **Sniper** attack with the candidate usernames list. Find the username that returns `Incorrect password` instead of `Invalid username`.

3. Replace the username with the discovered account and brute-force the password using a **Sniper** attack with the candidate passwords list.

4. Identify the password by the different response status code (`302`) and log in.

## Impact

An attacker can enumerate valid usernames and perform password brute-force attacks. If weak or predictable credentials are used, this can lead to unauthorized account access and account takeover.

## Mitigation

Implement generic login error messages that do not reveal whether a username exists.

Additional protections:
- Use rate limiting and account lockout mechanisms against repeated login attempts.
- Require strong, unique passwords.
- Implement multi-factor authentication (MFA).
- Monitor and block suspicious authentication activity.

## Tools
- Burp Suite
- Intruder

## Notes

This lab demonstrates how differences in application responses can reveal valid usernames. Comparing response messages, status codes, and response lengths allows attackers to identify valid accounts and then perform password brute-force attacks.