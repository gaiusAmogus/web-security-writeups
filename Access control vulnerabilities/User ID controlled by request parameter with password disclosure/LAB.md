# User ID controlled by request parameter with password disclosure

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-password-disclosure)  
**Category:** Access control  
**Difficulty:** APPRENTICE  

## Objective
To solve the lab, retrieve the administrator's password, then use it to delete the user `carlos`.

## Vulnerability
This lab has user account page that contains the current user's existing password, prefilled in a masked input.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps
1. Log in as `wiener:peter`, intercept the request and send it to Repeater.
2. Change the header to `GET /my-account?id=carlos` - this switched the account to `carlos`.
3. Change the header to `GET /my-account?id=administrator` - this switched to the `administrator` account, and in the password reset input, the value contains the password `fzgsgtu03iqfja73895u`.
4. In the browser, log in as `administrator:fzgsgtu03iqfja73895u` and delete the user `carlos`.

### Payload
```http
GET /my-account?id=administrator
```

### Shortcut
1. Intercept request -> Send to Repeater
2. Change id parameter to administrator
3. Read the password from the value attribute in the HTML
4. Log in as admin and delete carlos

## Impact
An attacker can hijack other users' accounts, including the administrator, gaining full control over the application.

## Mitigation
- Implement server-side access control checks.
- Never display passwords in plaintext in HTML.
- Use session-based identifiers instead of request parameters.

## Tools
- Burp Suite
- Repeater

## Notes
- The application trusts the id parameter without validating user permissions.
- The password was exposed in the value attribute of an input field.
- This lab shows how easy it is to escalate privileges through parameter manipulation.