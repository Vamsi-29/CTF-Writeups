# CTF-Style Practice Lab — IDOR Authorization Bypass

> **Status:** Self-created practice scenario  
> **Category:** Web Exploitation  
> **Technique:** Insecure Direct Object Reference (IDOR) / Broken Object-Level Authorization
>
> This is an intentionally self-created lab for practice and documentation. It is **not** an official CTF challenge, and it is not presented as a real challenge solve or achievement.

## Challenge / Context

The practice application is a small support portal with authenticated users and a ticket system. A normal user can view their own ticket through an endpoint similar to:

```text
GET /api/tickets/1001
```

The application correctly requires authentication, but the API does not verify that the authenticated user is authorized to access the requested ticket object.

The objective is to identify the authorization weakness and demonstrate how changing the object identifier exposes another user's ticket data inside the isolated lab.

## Reconnaissance / Analysis

Start by mapping the application normally rather than immediately modifying requests.

```bash
curl -i http://127.0.0.1:8080/
curl -i http://127.0.0.1:8080/login
```

After authenticating to the local lab account, inspect the requests made by the ticket page with Burp Suite or browser developer tools.

A request appears as:

```http
GET /api/tickets/1001 HTTP/1.1
Host: 127.0.0.1:8080
Cookie: session=<LAB_SESSION>
```

The response contains the ticket belonging to the logged-in user.

At this point the important observation is that the resource identifier is supplied directly in the URL:

```text
/api/tickets/1001
```

This makes object-level authorization a useful test case.

## Vulnerability / Technique

The issue is an **IDOR / Broken Object-Level Authorization** condition.

Authentication answers:

> "Who is the user?"

Authorization must additionally answer:

> "Is this user allowed to access this specific object?"

The vulnerable application checks the session but fails to enforce the second condition.

## Solution Steps

### 1. Capture the legitimate request

Use Burp Suite's HTTP history or the browser network panel and identify the ticket request:

```http
GET /api/tickets/1001 HTTP/1.1
```

Record the endpoint and object identifier. Do not reuse or publish real credentials or session cookies.

### 2. Test object-level authorization

Send the request to Burp Repeater and change only the object identifier:

```http
GET /api/tickets/1002 HTTP/1.1
```

Keep the same authenticated session.

A secure implementation should return an authorization failure such as:

```http
HTTP/1.1 403 Forbidden
```

In this practice scenario, the server instead returns a successful response containing a different user's ticket.

### 3. Confirm the behavior

Repeat with another nearby identifier:

```http
GET /api/tickets/1003 HTTP/1.1
```

If multiple objects can be accessed while using the same low-privileged session, the behavior is consistent with missing object-level authorization rather than a single malformed record.

### 4. Validate with curl

For a local lab, the same test can be reproduced from the command line. Use a placeholder for the lab session rather than storing a real credential in the repository:

```bash
curl -i \
  -H 'Cookie: session=<LAB_SESSION>' \
  http://127.0.0.1:8080/api/tickets/1002
```

The important comparison is between the response for the user's own object and the response for an object belonging to another user.

## Why the Exploit Works

The vulnerable authorization logic effectively behaves like:

```python
@app.get("/api/tickets/<int:ticket_id>")
def get_ticket(ticket_id):
    user = get_authenticated_user()
    if not user:
        return {"error": "unauthorized"}, 401

    return get_ticket_from_database(ticket_id)
```

The application verifies that a user is logged in, but it does not verify ownership of `ticket_id`.

A safer pattern is to enforce authorization as part of the database lookup:

```python
@app.get("/api/tickets/<int:ticket_id>")
def get_ticket(ticket_id):
    user = get_authenticated_user()
    if not user:
        return {"error": "unauthorized"}, 401

    ticket = get_ticket_for_user(ticket_id, user.id)
    if ticket is None:
        return {"error": "not found"}, 404

    return ticket
```

The authorization condition should be enforced server-side. Hiding IDs in the frontend or using less predictable identifiers is not a substitute for authorization checks.

## Result

The isolated practice lab demonstrates that changing a resource identifier can cross the intended user boundary when object-level authorization is missing.

No real account, credential, production system, or external target is involved in this scenario.

## Lessons Learned

- Authentication and authorization are separate security controls.
- Test every object identifier that appears in URLs, JSON bodies, and API parameters.
- Compare requests between two users when testing access-control boundaries.
- A `200 OK` response to another user's object is a strong signal for an authorization flaw.
- Server-side authorization must be enforced for every sensitive object access.
- Sequential IDs make testing easier to notice, but unpredictable IDs do not fix an authorization flaw.
- In real assessments, document the affected object type, access boundary, impact, and minimal proof required to reproduce the issue.

## Defensive Recommendations

1. Enforce object ownership or role-based authorization on every API request.
2. Scope database queries to the authenticated user's identity where appropriate.
3. Return consistent authorization/not-found behavior without leaking sensitive object existence.
4. Add automated authorization tests for horizontal and vertical privilege boundaries.
5. Review every API endpoint independently; fixing authorization in the frontend is insufficient.

---

**Practice classification:** Self-created CTF-style lab  
**Official challenge:** No  
**Claimed real-world result:** No
