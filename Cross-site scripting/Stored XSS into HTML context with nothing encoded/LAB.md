# Stored XSS into HTML context with nothing encoded

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/stored/lab-html-context-nothing-encoded)  
**Category:** XSS  
**Difficulty:** APPRENTICE

## Objective
To solve this lab, submit a comment that calls the `alert` function when the blog post is viewed.

## Vulnerability
This lab contains a stored cross-site scripting vulnerability in the comment functionality.

## Exploitation

### Steps
1. Entered the lab, there is a list of blog posts, clicked on a post.

2. At the bottom of the post there is a comment form, submitted it filled as follows:

```text
Comment:
<script>alert('test');</script>

Name:
Test

Email:
carlos.hacker@gmail.com

Website:
https://google.com

```

3. The alert did not display immediately, but the lab was solved - this suggests the code will execute when user (or admin) views the comment.

### Payload
```js
<script>alert('test');</script>
```

### Shortcut
1. Navigate to any blog post, scroll to the comment form, enter `<script>alert('test');</script>` in the comment field and fill in the other fields with any data, then submit.

## Impact
An attacker can inject persistent malicious scripts that execute in every user's browser who views the infected comment. This allows stealing session cookies, hijacking accounts, defacing pages, or redirecting users to malicious sites.

## Mitigation
Encode all user-supplied data before rendering it in HTML context - convert characters like `<`, `>`, `"`, `'`, and `&` to their HTML entities. Also implement Content Security Policy (CSP) and validate input on the server-side.

## Tools
- Browser

## Notes
The key difference from reflected XSS is that the payload is stored on the server and executed whenever the page is viewed. The alert didn't trigger immediately because the exploit targets other users viewing the comment, not the submitter. This is more dangerous than reflected XSS because it's persistent and affects all users.