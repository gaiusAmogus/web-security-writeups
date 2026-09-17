# Excessive trust in client-side controls

**Source:** [PortSwigger](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-excessive-trust-in-client-side-controls)  
**Category:** Business logic vulnerabilities  
**Difficulty:** Apprentice

## Objective
To solve the lab, buy a "Lightweight l33t leather jacket".

## Vulnerability
This lab doesn't adequately validate user input. You can exploit a logic flaw in its purchasing workflow to buy items for an unintended price. 

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps

1. Log in to your account. Find the **"Lightweight l33t leather jacket"** product on the home page. The product price is `$1337.00`.

2. Intercept the **Add to cart** request in Burp:
    ```text
    productId=1&redir=PRODUCT&quantity=1&price=133700
    ```
3. Modify the price in the request:
    - `price=0` - did not work (the product was not added).
    - `price=1` - worked: the product was added to the cart for `$0.01`.

4. Place the order - the user has `$100.00` in their account, the product costs `$0.01`, so the transaction succeeds. Lab solved.

### Payload
```text
productId=1&redir=PRODUCT&quantity=1&price=1
```

### Shortcut
1. Log in and intercept the **Add to cart** request for the leather jacket.
2. Change `price=133700` to `price=1`.
3. Forward the request and place the order.

## Impact
An attacker can manipulate the price sent by the client and purchase products at an arbitrary, near-zero cost. This results in financial loss and allows acquiring goods without paying the intended price.

## Mitigation
Never trust price values supplied by the client. Determine the price server-side from the product database using the product ID. Validate cart totals on the server before processing payment. Treat all client-supplied pricing data as untrusted input.

## Tools
- Burp Suite

## Notes
- The price is sent in the **Add to cart** request as a client-controlled parameter.
- `price=0` is rejected, but `price=1` is accepted.
- The server trusts the client-supplied price, enabling purchase at `$0.01`.
- Setting a very low but non-zero price bypasses simple zero-value checks.