# Multi-step process with no access control on one step

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-multi-step-process-with-no-access-control-on-one-step)  
**Category:** Access control  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, log in using the credentials `wiener:peter` and exploit the flawed access controls to promote yourself to become an administrator.

## Vulnerability
This lab has an admin panel with a flawed multi-step process for changing a user's role. You can familiarize yourself with the admin panel by logging in using the credentials `administrator:admin`.

## Exploitation

### Steps

1. Log in as `administrator:admin` and explore the admin panel, including analyzing requests in Burp Suite.

2. In the `Admin panel` there is a form to upgrade or downgrade users. Submit the upgrade form for the administrator and intercept the request.
Then a confirmation prompt appears asking if you are sure. After answering `yes`, a second request is sent with:

```http
POST /admin-roles HTTP/2
[...]
action=upgrade&confirmed=true&username=administrator
```

3. Log in as `wiener:peter` and send the confirmation request directly:

```http
POST /admin-roles HTTP/2
[...]
action=upgrade&confirmed=true&username=wiener
```

Success - user `wiener` promoted to administrator.

### Payload

```http
POST /admin-roles HTTP/2
[...]
action=upgrade&confirmed=true&username=wiener
```

### Shortcut

1. Log in as `administrator:admin` to identify the multi-step process.
2. Note that first step requires admin privileges, but confirmation step does not.
3. Log in as `wiener:peter`.
4. Send POST `/admin-roles` with `action=upgrade&confirmed=true&username=wiener`.

## Impact
An attacker with a regular account can bypass the admin check by skipping the first step and sending the confirmation request directly. This leads to privilege escalation - promoting themselves or other users to administrator without proper authorization.

## Mitigation

1. Apply access control checks on every step of a multi-step process, not just the first one.
2. Use server-side session state to track which steps have been completed and ensure users cannot skip steps.
3. Validate that the user performing the action has admin privileges before executing any sensitive operation.
4. Use CSRF tokens and nonce values to prevent request forgery and ensure the request comes from a legitimate session.

## Tools

- Burp Suite

## Notes

- The confirmation request `confirmed=true` did not check if the user was an admin.
- The first request was protected, but the second request was not.
- Always test all steps in a multi-step process for access control flaws.
- This vulnerability is a common mistake - developers assume that if the user reached step 2, they must have passed step 1.