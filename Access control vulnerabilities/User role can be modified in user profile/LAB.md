# User role can be modified in user profile

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-user-role-can-be-modified-in-user-profile)  
**Category:** Access control  
**Difficulty:** APPRENTICE  

## Objective
Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

## Vulnerability
This lab has an admin panel at `/admin`. It's only accessible to logged-in users with a roleid of 2.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

## Exploitation

### Steps
1. Logged in as `wiener:peter` and intercepted the request after page refresh. No visible vulnerabilities were found at this stage.

2. Navigated to `/admin` and received the response: `Admin interface only available if logged in as an administrator`.

3. Tried adding various parameters to the URL such as `/admin?roleid=2`, `/admin?role-id=2`, `/admin?id=2`, and `/admin?role=2` but none of them worked.

4. Sent the request to Repeater and attempted to add various headers:
- `X-Original-URL: /admin`
- `X-Forwarded-For: 127.0.0.1`
- `X-Forwarded-Host: 127.0.0.1`
- `X-Forwarded-Scheme: http`
- `X-Remote-IP: 127.0.0.1`
- `X-Remote-Addr: 127.0.0.1`
- `X-Real-IP: 127.0.0.1`
    
None of these bypass attempts were successful.

5. Filled out the email change form and intercepted the request. The request contained a JSON payload:
```json
{"email":"test@test.com"}
```

Modified it to:
```json
{"roleid":"2"}
```
and sent it, but it didn't work. Tried again with:
```json
{"roleid":2}
```
but received `"Method Not Allowed"` in response.

6. Sent the email change request to Repeater. The response contained a JSON object:
```json
{
    "username": "wiener",
    "email": "test@test.com",
    "apikey": "gD7khl6kFu3CntMxdcsC6eDQ1d147JSe",
    "roleid": 1
}
```

7. Sent the following JSON payload with the email change request:
```json
{
    "username": "wiener",
    "email": "test2@test.com",
    "apikey": "gD7khl6kFu3CntMxdcsC6eDQ1d147JSe",
    "roleid": 2
}
```
   
   The response confirmed that `roleid` changed to `2`. Navigated to `/admin` and deleted `carlos`.

### Payload
```json
{
    "username": "wiener",
    "email": "test2@test.com",
    "apikey": "gD7khl6kFu3CntMxdcsC6eDQ1d147JSe",
    "roleid": 2
}
```



### Shortcut
1. Log in to the application using credentials `wiener:peter`
2. Navigate to the email change form at `/my-account` and intercept the POST request to `/my-account/change-email` in Burp Suite
3. Modify the JSON payload by adding the `roleid` field with value `2` - ensure all existing fields from the response are included
4. Send the request and verify in the response that `roleid` has been changed to `2`
5. Access `/admin` and click the "Delete" button next to the user `carlos` to complete the lab


## Impact
Privilege escalation - a regular user can grant themselves administrative access.

## Mitigation
- Validate and filter input - only accept expected fields
- Never trust client-supplied role/permission data
- Enforce access control server-side

## Tools
- Burp Suite
- Repeater

## Notes
- Server blindly accepted all fields from JSON request body
- First attempts failed because the server expected the complete user object structure