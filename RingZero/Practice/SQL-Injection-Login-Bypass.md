# RingZero Practice — SQL Injection Login Bypass

> **Practice writeup:** This is a self-created CTF-style lab scenario. It is not an official RingZer0 challenge and is not presented as a challenge previously solved by the author.

## Challenge / Context

The lab provides a simple login form with a username and password field. The objective is to understand whether user-controlled input is being inserted directly into a SQL query and determine whether the authentication logic can be bypassed.

## Reconnaissance / Analysis

I started by submitting normal credentials and observing the application's response. I then tested the username field with a single quote:

```text
'
```

The application returned a database-related error instead of the normal login failure message. This suggested that the input was reaching the SQL query without proper parameterization.

## Vulnerability / Technique

The suspected vulnerability is **SQL Injection**.

The underlying insecure pattern can be represented as:

```sql
SELECT * FROM users
WHERE username = '<input>'
AND password = '<password>';
```

If the application concatenates input into the query, SQL syntax can be introduced through the username field.

## Exploitation / Solution

For the controlled lab, I tested a boolean condition that changes the authentication logic:

```text
' OR '1'='1' --
```

The payload closes the original string, adds an always-true condition, and comments out the remaining query.

A safer way to reason about the result is to compare:

```sql
username = '' OR '1'='1'
```

The second condition is always true, so the application's authentication query can return a user record when the vulnerable query construction is used.

## Result

The practice lab demonstrates an authentication bypass caused by unsafe SQL string concatenation.

Attack flow:

```text
Login input
   ↓
Single quote test
   ↓
SQL error / injection point
   ↓
Boolean-based payload
   ↓
Authentication logic altered
   ↓
Login bypass in vulnerable lab
```

## Lessons Learned

- A database error can be an early indicator of SQL injection.
- Authentication endpoints should never construct SQL statements by concatenating user input.
- Parameterized queries are the primary defense against SQL injection.
- Error messages should not expose unnecessary database implementation details.
- In real assessments, SQL injection testing must remain within the authorized scope.

## Defensive Fix

Use parameterized queries instead of string concatenation. The application should also apply appropriate server-side input handling and avoid returning raw database errors to users.

## Key Takeaway

The important technique is not simply knowing an SQLi payload. The useful workflow is **identify the injection point → understand the query context → demonstrate controlled impact → recommend parameterized queries**.
