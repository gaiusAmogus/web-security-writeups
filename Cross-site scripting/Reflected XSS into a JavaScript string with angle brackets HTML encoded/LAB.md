# Reflected XSS into a JavaScript string with angle brackets HTML encoded

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-javascript-string-angle-brackets-html-encoded)  
**Category:** XSS  
**Difficulty:** APPRENTICE

## Objective
To solve this lab, perform a cross-site scripting attack that breaks out of the JavaScript string and calls the `alert` function.

## Vulnerability
This lab contains a reflected cross-site scripting vulnerability in the search query tracking functionality where angle brackets are encoded. The reflection occurs inside a JavaScript string.

## Exploitation

### Steps
1. Enter the lab and type `' ; alert(1); //` into the search form.
2. Submit the search query.
3. The lab is solved.

### Payload
```html
' ; alert(1); //
```

### Shortcut
1. Go to search bar
2. Enter `' ; alert(1); //`
3. Press Enter


## Impact
An attacker can execute arbitrary JavaScript in the victim's browser, leading to:
- Session hijacking and cookie theft
- Credential phishing
- Unauthorized actions on behalf of the user
- Website defacement
- Malicious redirects
- Data theft

## Mitigation
- **Output encoding**: Properly encode user input when embedding in JavaScript strings (use backslash escaping)
- **Input validation**: Validate and sanitize user input
- **CSP**: Implement Content Security Policy
- **Use safe APIs**: Avoid constructing JavaScript strings with user input
- **Context-aware escaping**: Use different escaping rules for JavaScript context vs HTML context

## Tools
- Browser

## Notes
- Angle brackets HTML encoding does not prevent XSS in JavaScript contexts
- The single quote `'` breaks out of the JavaScript string
- `;` ends the current statement
- `alert(1)` executes the alert
- `//` comments out the rest of the JavaScript code to prevent syntax errors
- Always use context-appropriate encoding - HTML encoding is not sufficient for JavaScript contexts