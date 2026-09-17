# Insufficient workflow validation

**Source:** [PortSwigger](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-insufficient-workflow-validation)  
**Category:** Business logic vulnerabilities  
**Difficulty:** Practitioner

## Objective
To solve the lab, exploit this flaw to buy a "Lightweight l33t leather jacket".

## Vulnerability
This lab makes flawed assumptions about the sequence of events in the purchasing workflow. 

You can log in to your own account using the following credentials: `wiener:peter`

## Exploitation

### Steps
1. Log in as `wiener:peter`, add the **Lightweight l33t leather jacket** to the cart and try to place the order via `/cart/checkout`. The application rejects it due to insufficient funds. Try to manually trigger the finalization by changing `/cart/checkout` to `/cart/order` in the request - it returns `Not Found`.

2. Buy the cheapest product to see which endpoint is called after a successful checkout - we get `/cart/order-confirmation` in request.

3. Add the jacket back to the cart and send the finalization request, skipping the checkout:
    ```text
    GET /cart/order-confirmation?order-confirmed=true HTTP/2
    ```

4. The application accepts the order, the jacket is purchased. Lab solved.

### Payload
```text
GET /cart/order-confirmation?order-confirmed=true HTTP/2
```

### Shortcut
1. Log in as `wiener:peter` and add the leather jacket to the cart.
2. Complete a cheap purchase to discover `/cart/order-confirmation`.
3. Add the jacket to the cart again and request `GET /cart/order-confirmation?order-confirmed=true`.
4. The order is placed and the lab is solved.

## Impact
An attacker can bypass the checkout step by directly calling the order confirmation endpoint. This allows placing orders without proper payment validation, resulting in financial loss and purchase of items without sufficient funds.

## Mitigation
Enforce the full checkout and payment flow server-side. Do not expose a confirmation endpoint that finalizes orders without validating cart contents, totals, and payment. Use server-side state and session checks to ensure the order is only confirmed after a successful checkout. Reject requests that skip required steps.

## Tools
- Burp Suite
- Browser

## Notes
- `/cart/checkout` performs the payment validation.
- `/cart/order-confirmation?order-confirmed=true` finalizes the order directly.
- Calling the confirmation endpoint bypasses the balance check.