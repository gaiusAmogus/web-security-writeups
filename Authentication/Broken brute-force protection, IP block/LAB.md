# Broken brute-force protection, IP block

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block)  
**Category:** Authentication vulnerabilities  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, brute-force the victim's password, then log in and access their account page.

## Vulnerability
This lab is vulnerable due to a logic flaw in its password brute-force protection.

Your credentials: `wiener:peter`  
Victim's username: `carlos`  
[Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Exploitation

### Steps

1. Attempted to log in as `carlos` using incorrect passwords. After 2 failed login attempts, the login was blocked for 1 minute.

2. After waiting one minute, attempted to log in as `carlos` again using incorrect passwords. Intercepted the HTTP request, sent it to Repeater, and tried performing login attempts while changing the session. The account was still blocked after 2 failed attempts.

3. Attempted to log in as `carlos` once and then successfully logged in as `wiener:peter`. Discovered that this reset the failed login attempt counter, allowing another login attempt for `carlos`.

4. Sent the request to Intruder and configured a **Pitchfork** attack with a sequence of trying a password for `carlos` and then performing a valid login as `wiener:peter`.

5. Searched through the attack results for a request with HTTP status code `302` where payload 1 contained the `carlos` username.

6. Attempted to log in through the browser using `carlos:159753`, which was successful.

### Payload

```http
username=§carlos§&password=§password§
```

```http
username=wiener&password=peter
```

### Shortcut

1. Use the valid `wiener:peter` login to reset the failed login attempt counter after every incorrect `carlos` password attempt.

2. Configure a Burp Intruder **Pitchfork** attack:
   - Payload 1: `carlos` password list.
   - Payload 2: valid `wiener:peter` credentials to reset the counter.

3. Find the request returning HTTP status code `302` and log in as `carlos` with the discovered password.

## Impact

An attacker can bypass IP-based brute-force protection by using another valid account to reset the failed login attempt counter. This allows password brute-forcing against a victim account without triggering the protection mechanism.

## Mitigation

Implement stronger brute-force protection that cannot be reset by unrelated successful logins.

Additional protections:
- Track failed login attempts per account instead of only by IP address or session.
- Implement rate limiting with longer delays after repeated failures.
- Use multi-factor authentication (MFA).
- Monitor suspicious login patterns and repeated authentication attempts.

## Tools
- Burp Suite
- Repeater

## Notes

This lab demonstrates a logic flaw in brute-force protection. The application blocks repeated failed attempts but incorrectly resets the counter after a successful login to another account. By alternating between attacking the victim account and logging in with valid credentials, the brute-force protection can be bypassed.