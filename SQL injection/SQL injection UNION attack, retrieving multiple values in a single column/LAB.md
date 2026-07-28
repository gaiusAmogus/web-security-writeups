# SQL injection UNION attack, retrieving multiple values in a single column

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column)  
**Category:**  SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, perform a SQL injection UNION attack that retrieves all usernames and passwords, and use the information to log in as the `administrator` user.

## Vulnerability
This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response so you can use a UNION attack to retrieve data from other tables.

The database contains a different table called `users`, with columns called `username` and `password`.

## Exploitation

### Steps
1. Intercepted the HTTP request after selecting a product category.
2. Identified the vulnerable `category` GET parameter and sent the request to Repeater.
3. Modified the `category` parameter by appending `'+UNION+SELECT+NULL--` and observed an **Internal Server Error** response.
4. Determined that the payload `'+UNION+SELECT+NULL,NULL--` returned a valid response, indicating that the query contained two columns.
5. Replaced the first `NULL` with the string `'test'`, which resulted in an error. Replacing the second `NULL` with `'test'` returned a valid response, confirming that only the second column accepted string data.
6. Used the payload `'+UNION+SELECT+NULL,username||'~'||password+FROM+users--` to concatenate usernames and passwords into a single column. The response revealed multiple credentials, including those of the `administrator` user.
7. Logged in successfully as the `administrator` user using the retrieved credentials.

### Payload
```sql
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

### Shortcut

1. Find the number of columns:
```sql
'+UNION+SELECT+NULL,NULL--
```
2. Identify the column that accepts text data by replacing `NULL` with a string.
3. Extract usernames and passwords using concatenation:
```sql
'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```
4. Use the retrieved `administrator` credentials to log in.

## Impact
An attacker can retrieve sensitive information even when only a single column is available for displaying text. By concatenating multiple values into one column, it is still possible to disclose usernames, passwords, and other confidential data, potentially leading to account compromise and privilege escalation.

## Mitigation
Use parameterized queries (prepared statements) for all database operations. Store passwords securely using strong password hashing algorithms instead of plaintext, and avoid exposing database errors or query results that could aid an attacker.

## Tools
- Burp Suite
- Repeater

## Notes
When only one column supports string data, multiple database values can be combined into a single output using SQL string concatenation. This technique allows attackers to extract multiple pieces of information despite output limitations imposed by the application.