# Username enumeration via response timing

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing)  
**Category:** Authentication vulnerabilities  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

## Vulnerability
This lab is vulnerable to username enumeration using its response times. 

Your credentials: `wiener:peter`  
[Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)  
[Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Hint
To add to the challenge, the lab also implements a form of IP-based brute-force protection. However, this can be easily bypassed by manipulating HTTP request headers.

## Exploitation

### Steps

1. Logged in using `test:test`, intercepted the HTTP request, and sent it to Repeater.

2. Attempted to log in using `wiener` with an incorrect password, then tried logging in with usernames from the candidate usernames list to compare the responses. After 5 login attempts, the application blocked further login attempts for 30 minutes.

3. Added the `X-Forwarded-For: 192.168.0.1` header, which reset the login attempt counter.

4. After several tests, no visible differences were observed in Repeater, so the request was sent to Intruder.

5. Configured a **Pitchfork** attack using the candidate usernames list while also changing the last octet of the IP address.

```http
X-Forwarded-For: 192.168.0.§3§
```

```http
username=§test§&password=test
```

6. Observed a significant difference between **Response received** and **Response completed** for the username `carlos`.

7. Configured a second **Pitchfork** attack using the username `carlos`, changing the IP address and using the candidate passwords list. Changed the IP range to `192.168.1.§1§` to avoid reusing IP addresses from the previous attack.

8. None of the results indicated a successful login. Returned to step 5, but instead of using the password `test`, changed it to:

```http
username=§test§&password=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

Also changed the IP range again to avoid duplicates and set **Maximum concurrent requests** to `1`.

9. Observed a significantly higher response time for the username `arcsight`. Performed the password attack against this username in the same way as in step 7.

10. Observed HTTP status code `302` for the password `mobilemail`. Logged in through the browser using these credentials, which was successful.

### Payload

Username enumeration:

```http
POST /login HTTP/2
X-Forwarded-For: 192.168.0.§2§

username=§1§&password=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

Password brute-force:

```http
POST /login HTTP/2
X-Forwarded-For: 192.168.1.§2§

username=arcsight&password=§1§
```

### Shortcut

1. Bypass the IP-based rate limit by changing the `X-Forwarded-For` header on every request.

2. Use a **Pitchfork** attack with a long invalid password and compare **Response received** vs **Response completed** to identify the valid username.

3. Perform another **Pitchfork** attack against the discovered username using the candidate passwords list and changing the IP address on every request. Log in using the credentials that return HTTP status code `302`.

## Impact

An attacker can enumerate valid usernames by measuring subtle differences in response times. Combined with bypassing IP-based brute-force protection, this allows credentials to be brute-forced and user accounts to be compromised.

## Mitigation

Use identical processing logic and response times for valid and invalid usernames.

Additional protections:
- Implement rate limiting that cannot be bypassed using client-controlled headers.
- Do not trust headers such as `X-Forwarded-For` unless they are added by a trusted reverse proxy.
- Use generic authentication responses.
- Require multi-factor authentication (MFA).
- Monitor and block suspicious authentication activity.

## Tools

- Burp Suite
- Repeater
- Intruder

## Notes

This lab demonstrates that response timing differences can reveal valid usernames even when the application returns identical error messages. It also shows how relying on the client-controlled `X-Forwarded-For` header for IP-based rate limiting allows brute-force protection to be bypassed completely.