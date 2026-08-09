# User role controlled by request parameter

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-user-role-controlled-by-request-parameter)  
**Category:** Access control  
**Difficulty:** APPRENTICE  

## Objective
Solve the lab by accessing the admin panel and using it to delete the user carlos.

## Vulnerability
This lab has an admin panel at /admin, which identifies administrators using a forgeable cookie.

You can log in to your own account using the following credentials: wiener:peter

## Exploitation

### Steps

1. Entered the lab and tried adding `/admin` to the URL, but access was denied.

2. Went to the login page, attempted to log in using `test:test`, and intercepted the request. There were no obvious vulnerabilities visible in the request.

3. Logged in using the valid credentials `wiener:peter` and intercepted the request generated when refreshing the page after logging in. The following cookie was visible in the request:
   `Cookie: Admin=false;`

4. Changed the cookie value to:
   `Cookie: Admin=true;`

   and sent the request. This unlocked additional menu options, including `Admin panel`. When accessing the admin panel, the `Admin=true` cookie had to remain present in subsequent requests; otherwise, access was denied.

5. Entered the `Admin panel` and deleted the user `carlos`, making sure that every request continued to contain `Admin=true` in the cookie.

### Payload

```text
Cookie: Admin=true;
```

### Shortcut

1. Log in using `wiener:peter`.

2. Intercept an authenticated request and change:
   `Admin=false`

   to:
   `Admin=true`

3. Access `/admin` while keeping `Admin=true` in the cookie, then delete the user `carlos`.

## Impact

An attacker can modify the client-side `Admin` cookie and gain administrative privileges without proper authorization. This allows unauthorized access to restricted functionality, such as the admin panel and user management actions.

## Mitigation

Authorization decisions should never rely on client-controlled values such as cookies or request parameters.

The application should verify the user's role and permissions on the server side for every privileged request. Administrative functionality should only be accessible when the authenticated server-side session confirms that the user has the required privileges.

## Tools

- Burp Suite

## Notes

The vulnerability exists because the application trusts the value of the `Admin` cookie supplied by the client. Since the user can modify this value, changing it from `false` to `true` is enough to bypass the access control mechanism.
