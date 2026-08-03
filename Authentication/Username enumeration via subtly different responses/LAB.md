# Username enumeration via subtly different responses

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses)  
**Category:** Authentication vulnerabilities  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

## Vulnerability
This lab is subtly vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

[Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)  
[Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Exploitation

### Steps
1. Attempted to log in using `test:test` and intercepted the HTTP request.

2. Sent the request to Burp Intruder and configured a **Sniper** attack on the `username` parameter. Added usernames from the candidate usernames wordlist and started the attack.

```http
username=§test§&password=test
```

3. Compared the responses but found no obvious differences that could identify a valid username.

4. Added the phrase `Invalid username or password.` to **Grep - Match**. The results showed that the response for the username `academico` was missing the final `.` character, indicating that it was likely a valid username.

5. Repeated the attack using the identified username and configured a **Sniper** attack on the `password` parameter with the candidate passwords wordlist.

```http
username=academico&password=§test§
```

6. The password `555555` returned an HTTP status code `302`, indicating a successful login. Verified the credentials by logging in through the browser as `academico:555555`.

### Payload

**Username enumeration:**

```http
username=§test§&password=test
```

**Password brute-force:**

```http
username=academico&password=§test§
```

**Discovered credentials:**

```text
academico:555555
```

### Shortcut

1. Send the login request to Burp Intruder and perform a **Sniper** attack on the `username` parameter using the candidate usernames wordlist.
2. Use **Grep - Match** to detect the response that is missing the final `.` in the `Invalid username or password.` message, revealing the valid username.
3. Perform a **Sniper** attack on the `password` parameter using the discovered username. A successful login is indicated by an HTTP status code `302`.

## Impact

Username enumeration allows attackers to identify valid accounts by observing subtle differences in authentication responses. Combined with password brute-force attacks, this can lead to unauthorized access to user accounts and compromise sensitive information.

## Mitigation

Return identical authentication responses regardless of whether the username or password is incorrect. Ensure that response content, formatting, status codes, and response times are consistent. Implement rate limiting, account lockout after repeated failed attempts, and Multi-Factor Authentication (MFA) to reduce the effectiveness of brute-force attacks.

## Tools

- Burp Suite
- Intruder

## Notes

This lab demonstrates how even a minor difference in an authentication response can disclose whether a username exists. Burp Suite's **Grep - Match** feature makes it easy to identify these subtle differences, while Burp Intruder can automate both username enumeration and password brute-force attacks.