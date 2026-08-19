# Stored XSS into anchor href attribute with double quotes HTML-encoded

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-href-attribute-double-quotes-html-encoded)  
**Category:** XSS  
**Difficulty:** APPRENTICE

## Objective
To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.

## Vulnerability
This lab contains a stored cross-site scripting vulnerability in the comment functionality.

## Exploitation

### Steps
1. Enter the lab and go to the blog post with the comment form. Submit a test comment with the `Website` field filled in. After submission, observe that the author's `Name` becomes a link.

```html
<a id="author" href="https://google.com">Test</a>
```

2. Submit another comment, but this time add `javascript:alert(1)` in the `Website` field. The comment is sent, and the lab is solved.

### Payload
```html
javascript:alert(1) 
```

### Shortcut
1. Go to comment section, enter `javascript:alert(1)` in `Website` field and submit comment

## Impact
An attacker can execute arbitrary JavaScript in the victim's browser when they click the author's name, leading to:
- Session hijacking and cookie theft
- Credential phishing
- Unauthorized actions on behalf of the user
- Website defacement
- Malicious redirects

## Mitigation
- **Protocol validation**: Only allow HTTP/HTTPS protocols, block `javascript:`, `data:`, `vbscript:`
- **Output encoding**: Context-aware encoding for HTML attributes
- **CSP**: Implement Content Security Policy to restrict script execution
- **Input sanitization**: Use libraries like DOMPurify to sanitize URLs
- **Safe URL handling**: Parse and validate URLs with proper libraries

## Tools
- Browser

## Notes
- Double quote encoding prevents attribute injection but not `javascript:` protocol abuse
- The `javascript:` pseudo-protocol in `href` executes JavaScript on click
- Always validate URL schemes in user-supplied links
- Context-aware sanitization is essential for security