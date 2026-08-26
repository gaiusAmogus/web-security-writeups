# URL-based access control can be circumvented

**Source:** [PortSwigger](https://portswigger.net/web-security/access-control/lab-url-based-access-control-can-be-circumvented)  
**Category:** Access control  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, access the admin panel and delete the user `carlos`.

## Vulnerability
This website has an unauthenticated admin panel at `/admin`, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the `X-Original-URL` header.

## Exploitation

### Steps
1. Enter the lab, the `Admin Panel` tab is visible. Attempting to access it returns `"Access denied"`.

2. Capture the `/admin` request and send it to Repeater. Add `X-Original-URL: /admin` - returns `"Access denied"`.

3. Change the request from `GET /admin HTTP/2` to `GET / HTTP/2` while keeping `X-Original-URL: /admin` - successfully logs into the admin panel.

4. Open the response in the browser and attempt to delete user `carlos` - returns `"Access denied"`.

5. Return to Repeater with the knowledge that the deletion path is `/admin/delete?username=carlos`. Send a request with `GET / HTTP/2` and `X-Original-URL: /admin/delete?username=carlos` - returns `"Missing parameter 'username'"`.

6. Attempt sending a request with two additional headers:

```http
X-Original-Url: /admin/delete
X-Original-Url-Args: username=carlos
```

Still returns "Missing parameter 'username'".

7. Attempt sending a request with `X-Original-Url: /admin/delete` and append `username=carlos` at the very bottom of the request body - returns status 302 Found.

8. Access `/admin` again as before to verify if the user was deleted - user `carlos` is no longer present.


### Payload
```http
GET / HTTP/2
Host: 0ad6000b03f63ec180df711f004400a0.web-security-academy.net
X-Original-Url: /admin/delete
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

username=carlos
```

### Shortcut

1. Capture any request to the lab in Burp Suite.

2. Send it to Repeater.

3. Modify the request to use the `X-Original-URL` header pointing to `/admin` with the base path set to `/` to access the admin panel.

4. To delete the user, change the request path to `/`, set `X-Original-Url: /admin/delete`, and include `username=carlos` in the request body while keeping the GET method.

## Impact
An attacker can bypass front-end access controls and directly access administrative endpoints, allowing unauthorized actions such as deleting users, modifying content, or accessing sensitive data without proper authentication or authorization checks.

## Mitigation
1. Implement access control on the back-end - never rely solely on front-end filters or reverse proxy rules to restrict access.

2. Disable or restrict the X-Original-URL header in the application framework if not explicitly needed, or validate that the path matches the original request URL.

3. Use proper session management and role-based access control (RBAC) - all administrative endpoints should verify the user's session and permissions before processing any request.

4. Canonicalize URLs before applying access control rules to prevent path traversal and header manipulation attacks.

5. Implement defense in depth - combine network-level filtering with application-level authorization.

## Tools
- Burp Suite
- Repeater

## Notes
- The X-Original-URL header is processed by the back-end framework (e.g., Spring, Tomcat) and overrides the path from the request line.
- The front-end system only checks the request line path (e.g., /), allowing the attacker to "smuggle" the admin path through the header.
- For the deletion request, using GET with query parameters in the X-Original-URL header does not work because the back-end expects username as a POST parameter.
- Changing the method to POST and placing username=carlos in the request body successfully deletes the user.
- The 302 redirect after the POST request is expected behavior - it redirects back to the admin panel after successful deletion.
- This lab demonstrates that access control must be enforced on the server side, not just at the front-end or proxy level.