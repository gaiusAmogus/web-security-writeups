# 2FA simple bypass

**Source:** [PortSwigger](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-simple-bypass)  
**Category:** Authentication vulnerabilities  
**Difficulty:** APPRENTICE

## Objective
To solve the lab, access Carlos's account page.

## Vulnerability
This lab's two-factor authentication can be bypassed. You have already obtained a valid username and password, but do not have access to the user's 2FA verification code.

Your credentials: `wiener:peter`
Victim's credentials `carlos:montoya`

## Exploitation

### Steps

1. Logged in using the provided credentials `wiener:peter`.

2. Opened the email client and retrieved the 2FA verification code (`0479`). Completed the login process.

3. Observed that, after successful authentication, the account page URL was:

```text
/my-account?id=wiener
```

4. Logged out and signed in using the victim's credentials `carlos:montoya`.

5. When redirected to the 2FA verification page (`/login2`), manually changed the URL to:

```text
/my-account?id=carlos
```

6. The application granted direct access to Carlos's account page without validating the second authentication factor, successfully solving the lab.

### Payload

Authenticated account page:

```text
/my-account?id=wiener
```

2FA bypass:

```text
/my-account?id=carlos
```

### Shortcut

1. Log in as `carlos:montoya`.
2. When redirected to `/login2`, replace the URL with `/my-account?id=carlos`.
3. Access Carlos's account without completing the 2FA challenge.

## Impact

An attacker with valid username and password credentials can completely bypass the second authentication factor by directly accessing authenticated resources. This defeats the purpose of 2FA and allows unauthorized access to protected accounts.

## Mitigation

Ensure that access to authenticated resources is granted only after successful completion of all authentication steps, including 2FA.

Additional protections:
- Verify the user's 2FA status on the server before serving protected resources.
- Prevent direct access to authenticated endpoints until the authentication flow is complete.
- Store the authentication state securely on the server instead of relying solely on client-side navigation.
- Perform authorization checks on every request.

## Tools

- Web Browser

## Notes

This lab demonstrates a flawed authentication flow where the application validates the username and password but fails to verify that the user has successfully completed the second authentication factor before granting access to protected resources.