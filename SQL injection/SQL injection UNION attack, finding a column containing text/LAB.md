# SQL injection UNION attack, finding a column containing text

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text)  
**Category:**  SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, perform a SQL injection UNION attack that returns an additional row containing the value provided. This technique helps you determine which columns are compatible with string data.

## Vulnerability
This lab contains a SQL injection vulnerability in the product category filter. The results from the query are returned in the application's response, so you can use a UNION attack to retrieve data from other tables. To construct such an attack, you first need to determine the number of columns returned by the query. You can do this using a technique you learned in a previous lab. The next step is to identify a column that is compatible with string data.

The lab will provide a random value that you need to make appear within the query results. 

## Exploitation


### Steps
1. Intercepted the HTTP request after selecting a product category.
2. Identified the vulnerable `category` GET parameter and sent the request to Repeater.
3. Modified the `category` parameter by appending `'+UNION+SELECT+NULL--` and observed an **Internal Server Error** response.
4. Determined that the payload `'+UNION+SELECT+NULL,NULL,NULL--` returned a valid response, indicating that the query contained three columns.
5. Replaced each `NULL` value with the string `'xfOIaD'` one at a time. Invalid columns resulted in an **Internal Server Error**, while replacing the second `NULL` caused the value `'xfOIaD'` to be displayed in the application's response, confirming that the second column accepts string data.


### Payload
```text
'+UNION+SELECT+NULL,'xfOIaD',NULL--
```

### Shortcut

1. Determine the number of columns:
```sql
'+UNION+SELECT+NULL,NULL,NULL--
```
2. Test each column with a string value to find which one accepts text:
```sql
'+UNION+SELECT+NULL,'xfOIaD',NULL--
```
3. Confirm that the displayed value appears in the response, identifying the text-compatible column.

## Impact
An attacker can identify which columns accept string data, allowing the construction of more advanced UNION-based SQL injection payloads to extract sensitive information such as usernames, passwords, or other database contents.

## Mitigation
Use parameterized queries (prepared statements) instead of concatenating user input into SQL queries. Validate user input where appropriate and suppress detailed database error messages to prevent attackers from gathering information about the database structure.

## Tools
- Burp Suite
- Repeater

## Notes
Identifying both the number of columns and which columns accept string data is essential for successful UNION-based SQL injection. Displaying a controlled string in the application's response confirms that data from arbitrary SQL queries can be returned to the user.