# WT authentication bypass via weak signing key

**Source:** [PortSwigger](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-weak-signing-key)  
**Category:** JWT  
**Difficulty:** Practitioner

## Objective
<!-- What needs to be achieved? -->

## Vulnerability
This lab uses a JWT-based mechanism for handling sessions. It uses an extremely weak secret key to both sign and verify tokens. This can be easily brute-forced using a [wordlist of common secrets](https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list).

To solve the lab, first brute-force the website's secret key. Once you've obtained this, use it to sign a modified session token that gives you access to the admin panel at `/admin`, then delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps
1. Log in as `wiener:peter` and intercept the request in Burp. The `session` cookie contains a JWT with the `HS256` algorithm and payload `{"sub":"wiener"}`.

2. Crack the HMAC key by brute-force in JWT Editor (**Attack → Weak HMAC secret**). Found key:
    ```text
    secret1
    ```

3. Add the key in **JWT Editor Keys** as a symmetric key (Base64: `c2VjcmV0MQ`).

4. Change the payload to `{"sub":"administrator"}` and sign the token with the `secret1` key (HMAC-SHA256) via **Sign**.

5. Paste the new token into the `session` cookie and send:
    ```text
    GET /admin HTTP/2
    ```
    The response returns the admin panel.

6. Delete the user `carlos`:
    ```text
    GET /admin/delete?username=carlos HTTP/2
    ```
    Response `302 Found` — lab solved ✅


### Payload
```text
eyJraWQiOiI0MzRiMWIzZi0yZTdkLTQ4MDUtOGZmNi1lZjVhNWM0YTQ4YzgiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJwb3J0c3dpZ2dlciIsImV4cCI6MTc4OTY2NDMzNSwic3ViIjoiYWRtaW5pc3RyYXRvciJ9.[HMAC-SHA256 signature with secret1]
```

### Shortcut
1. Log in as `wiener:peter` and intercept the `session` JWT.
2. Brute-force the weak HMAC secret in JWT Editor (`secret1`).
3. Change `sub` to `administrator` and re-sign with `secret1`.
4. Send `GET /admin` with the new token.
5. Send `GET /admin/delete?username=carlos`.

## Impact
An attacker can crack a weak HMAC secret, forge a JWT with arbitrary claims, and impersonate any user, including `administrator`. This leads to privilege escalation, access to the admin panel, and execution of privileged actions such as deleting users.

## Mitigation
Use strong, randomly generated HMAC secrets with sufficient entropy. Never use guessable or dictionary-based keys. Rotate signing keys regularly. Enforce a strict allowlist of accepted algorithms and always verify the signature server-side. Validate all claims (`iss`, `exp`, `sub`) and bind the token to the session.

## Tools
- Burp Suite
- Repeater
- JWT Editor

## Notes
- The JWT uses `HS256` with a weak, guessable secret.
- JWT Editor's **Weak HMAC secret** attack recovers `secret1`.
- Re-signing the token with the cracked key allows changing `sub` to `administrator`.
- The forged token grants access to `/admin`.
- Deleting `carlos` returns `302 Found`.