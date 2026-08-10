# User ID controlled by request parameter with data leakage in redirect

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-user-id-controlled-by-request-parameter-with-data-leakage-in-redirect)  
**Category:** Access control  
**Difficulty:** APPRENTICE  

## Objective
To solve the lab, obtain the API key for the user `carlos` and submit it as the solution.

## Vulnerability
This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps
1. Logged in as `wiener:peter`, intercepted the request and sent it to Repeater.
2. Changed the parameter to `GET /my-account?id=carlos HTTP/2` and sent it. The server returned the account page logged in as `carlos` and displayed the API key.
3. Submitted the API key to the solution form.

### Payload
`GET /my-account?id=carlos HTTP/2`

### Shortcut
1. Log in as `wiener:peter`
2. Change `id` parameter to `carlos` in request with Repeater
3. Copy API key from response and submit

## Impact
An attacker can access any user's account data, including sensitive information such as API keys, by simply modifying the `id` parameter in the URL. This could lead to account takeover, data breach, and unauthorized access to protected resources.

## Mitigation
- Never rely on user-supplied input (like URL parameters) to determine which user's data to display.
- Always use server-side session data to identify the authenticated user.
- Implement proper access controls that verify the authenticated user has permission to access the requested resource.
- Use indirect references or encrypted identifiers instead of predictable usernames or sequential IDs.

## Tools
- Burp Suite
- Repeater

## Notes
- This lab demonstrates a classic Insecure Direct Object Reference (IDOR) vulnerability.
- Even though the lab description mentions data leakage in redirects, in this case the server returned a 200 OK response with the data directly.
- The vulnerability exists because the server does not validate whether the requested `id` matches the authenticated user's session.
- Simple parameter manipulation can lead to serious security breaches - never trust client-side data.