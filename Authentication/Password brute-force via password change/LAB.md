# Password brute-force via password change

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-brute-force-via-password-change)  
**Category:** Authentication vulnerabilities  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, use the list of candidate passwords to brute-force Carlos's account and access his "My account" page.

## Vulnerability
This lab's password change functionality makes it vulnerable to brute-force attacks.

Your credentials: `wiener:peter`  
Victim's username: `carlos`  
[Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

## Exploitation

### Steps

1. Logged in using `wiener:peter` and opened the password change page. Entered an incorrect current password and set the new password to `test`. The application logged the user out and blocked further login attempts for one minute.

2. After one minute, logged in again using `wiener:peter` and entered the correct current password, but intentionally used two different new passwords. Instead of being logged out, the application returned the message `New passwords do not match`. Intercepted the request and sent it to Repeater. Sent the request several times to verify whether the session would be terminated. In this case, the session remained active, allowing further testing using this request.

3. Sent the request with an incorrect current password while still using two different new passwords. The application returned the message `Current password is incorrect`, showing a clear difference compared to the case where the current password is correct but the new passwords do not match. Tested multiple requests to verify that the session was not terminated.

4. The request contained the following body:

```http
username=wiener&current-password=peter2&new-password-1=test&new-password-2=test2
```

Sent the request to Intruder and modified it to:

```http
username=carlos&current-password=§test§&new-password-1=test&new-password-2=test2
```

Configured a **Sniper** attack using the candidate passwords list. In **Grep - Match**, added the string `New passwords do not match`. This made it possible to identify which current password was correct. 

The attack returned a match for the password `zxcvbn`.

5. Attempted to log in using `carlos:zxcvbn`, which was successful.

### Payload

Original request:

```http
username=wiener&current-password=peter2&new-password-1=test&new-password-2=test2
```

Intruder request:

```http
username=carlos&current-password=§test§&new-password-1=test&new-password-2=test2
```

Grep - Match:

```text
New passwords do not match
```

### Shortcut

1. Intercept a password change request where the two new passwords are different.
2. Replace the username with `carlos` and brute-force the `current-password` parameter using a **Sniper** attack.
3. Use `New passwords do not match` as the success indicator, then log in using the discovered password.

## Impact

An attacker can brute-force another user's current password through the password change functionality by observing differences in application responses. This can lead to unauthorized account access without triggering traditional login brute-force protections.

## Mitigation

Ensure that password change functionality returns identical responses regardless of whether the current password is correct or incorrect.

Additional protections:
- Apply the same brute-force protections to password change endpoints as to the login page.
- Require re-authentication or multi-factor authentication (MFA) before changing a password.
- Rate limit password change attempts.
- Avoid exposing response differences that reveal whether the current password is valid.

## Tools

- Burp Suite
- Repeater
- Intruder

## Notes

This lab demonstrates that password change functionality can become an alternative brute-force target when it leaks information through different error messages. By intentionally submitting mismatched new passwords, it is possible to distinguish between valid and invalid current passwords, making automated password guessing feasible.