# Inconsistent security controls

**Source:** [PortSwigger](https://portswigger.net/web-security/logic-flaws/examples/lab-logic-flaws-inconsistent-security-controls)  
**Category:** Business logic vulnerabilities  
**Difficulty:** Apprentice

## Objective
To solve the lab, access the admin panel and delete the user `carlos`.

## Vulnerability
This lab's flawed logic allows arbitrary users to access administrative functionality that should only be available to company employees. 

## Exploitation

### Steps
1. Access the lab and go to the registration form. We see a very useful piece of information there - `If you work for DontWannaCry, please use your @dontwannacry.com email address`.

2. Try to create an account with a `@dontwannacry.com` email address, but it does not log us in - it requires confirmation via a link from the email.

3. Register a new account using an email address from the exploit server (`attacker@exploit-0ac200ef03a9fbf880b75c1c01280049.exploit-server.net`), confirm the registration via the link from the **Email client**, and then log in to the newly created account.

4. Go to **My account** and use the **Update email** function to change the email address to the company domain `@dontwannacry.com`.
    The application accepts the change **without verifying** the new email address - this is the logic flaw. We now get the ability to access the admin panel.

5. Delete the user `carlos` from the admin panel. Lab solved.

### Shortcut
1. Register an account using an exploit server email address.
2. Confirm the registration via **Email client** and log in.
3. Use **Update email** to change the email to `xxx@dontwannacry.com`.
4. Access the admin panel and delete `carlos`.

## Impact
An attacker can bypass email domain restrictions by registering with an arbitrary email and then changing it to a company domain without verification. This grants access to administrative functionality, allowing privileged actions such as deleting arbitrary users.

## Mitigation
Verify the new email address before applying the change, using a confirmation link sent to the new address. Enforce domain restrictions consistently at every step. Do not trust the current email value when granting role-based access; re-check the verified email server-side. Bind administrative privileges to server-side roles, not to email domains.

## Tools
- Browser

## Notes
- The registration form hints that `@dontwannacry.com` addresses belong to company employees.
- Registration with a company domain requires email confirmation.
- **Update email** accepts a company domain **without verification**.
- Changing the email grants admin panel access.