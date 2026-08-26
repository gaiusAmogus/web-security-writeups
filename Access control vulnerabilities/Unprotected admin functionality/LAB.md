# Unprotected admin functionality

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-unprotected-admin-functionality)  
**Category:** Access control  
**Difficulty:** APPRENTICE

## Objective
Solve the lab by deleting the user carlos.

## Vulnerability
This lab has an unprotected admin panel.

## Exploitation

### Steps

1. Searched for hidden or non-obvious paths by manually entering common admin panel locations:
   - `/login`
   - `/wp-admin`
   - `/wp-panel`
   - `/wp-login`
   - `/panel`
   - `/admin`
   - `/administrator`
   - `/adminpanel`
   - `/admin-panel`
   - `/administratorpanel`
   - `/administrator-panel`

2. The `/administrator-panel` path was accessible. Opened the admin panel and deleted the `carlos` user.

### Payload

Discovered admin panel:

```text
/administrator-panel
```

### Shortcut

1. Open the accessible `/administrator-panel`.
2. Delete the `carlos` user.

## Impact

An attacker can access administrative functionality without authentication or authorization. This can lead to unauthorized actions such as deleting users, modifying application settings, or gaining complete control over the application.

## Mitigation

Restrict access to administrative functionality using proper authentication and authorization checks.

Additional protections:
- Deny access to administrative endpoints for unauthenticated users.
- Enforce role-based access control (RBAC) on every administrative request.
- Avoid relying on hidden or unpredictable URLs as a security mechanism.
- Monitor and log access to administrative interfaces.

## Tools

- Browser

## Notes

This lab demonstrates that hiding an administrative interface is not a security control. If administrative functionality is accessible without authorization checks, an attacker only needs to discover the correct URL to gain full administrative access.