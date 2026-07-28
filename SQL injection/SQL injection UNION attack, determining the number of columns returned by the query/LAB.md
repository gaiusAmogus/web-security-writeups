# SQL injection UNION attack, determining the number of columns returned by the query

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns)  
**Category:**  SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, determine the number of columns returned by the query by performing a SQL injection UNION attack that returns an additional row containing null values.

## Vulnerability
This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. The first step of such an attack is to determine the number of columns that are being returned by the query. You will then use this technique in subsequent labs to construct the full attack.

## Exploitation

### Steps
1. Intercepted the HTTP request after selecting a product category.
2. Identified the vulnerable `category` GET parameter and sending request to the Repeater.
3. Modified the `category` parameter by appending `'+UNION+SELECT+NULL--` and observed an **Internal Server Error** response.
4. Gradually added additional `NULL` values until the application returned a valid response without errors, revealing the correct number of columns.


### Payload
```sql
'+UNION+SELECT+NULL,NULL,NULL--
```

### Shortcut
Use a `UNION SELECT` with increasing `NULL` values until the response is valid. The number of `NULL` values used in the successful payload equals the number of columns returned by the original query.
```sql
'+UNION+SELECT+NULL,NULL,NULL--
```

## Impact
An attacker can determine the number of columns returned by the original query, which is a required step for constructing successful UNION-based SQL injection attacks to extract data from other database tables.

## Mitigation
Use parameterized queries (prepared statements) instead of concatenating user input into SQL queries. In addition, avoid exposing detailed database errors to users, as they can reveal information useful for exploitation.

## Tools
- Burp Suite
- Repeater

## Notes
Determining the correct number of columns is a prerequisite for UNION-based SQL injection. Using `NULL` values is a common technique because they are compatible with most SQL data types, making it easier to identify the required column count.