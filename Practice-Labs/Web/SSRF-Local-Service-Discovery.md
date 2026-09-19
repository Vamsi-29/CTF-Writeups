# Web Exploitation Practice — SSRF Local Service Discovery

**Category:** Web Exploitation  
**Technique:** Server-Side Request Forgery (SSRF)  
**Difficulty:** Medium  
**Type:** Self-Created Practice Lab

> This is an original CTF-style practice scenario created for learning. It is not an official CTF challenge and does not represent a real solved challenge, flag, ranking, or achievement.

## Challenge / Context

A local training application provides a URL preview feature. The server accepts a user-supplied URL, fetches the resource from the server side, and returns a small portion of the response.

The lab contains a second HTTP service bound only to the loopback interface. The objective is to determine whether the preview feature can be abused to make the application request a resource that the analyst's browser cannot access directly.

The entire scenario is intended to run locally in an isolated practice environment.

## Reconnaissance / Analysis

First, interact with the URL preview feature using a normal external or local URL and observe the request/response behavior.

For example, a request may look conceptually like:

```http
POST /preview HTTP/1.1
Content-Type: application/x-www-form-urlencoded

url=http://example.test/
```

The important observation is that the application, rather than the browser, performs the HTTP request.

A safe local test can use a service intentionally started for the lab:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

If the preview endpoint can retrieve content from this loopback service, the application is acting as a server-side HTTP client.

## Vulnerability / Technique

The weakness is **Server-Side Request Forgery (SSRF)**: untrusted user input controls the destination of a request made by the server.

The security boundary is important. A browser may be unable to reach a service bound to `127.0.0.1`, while the vulnerable application can reach it because both the application and the internal service run on the same host or network.

The attack model is:

```text
User-controlled URL
        ↓
Preview endpoint
        ↓
Server-side HTTP request
        ↓
Loopback / internal service
        ↓
Response returned to user
```

## Solution Steps

### 1. Establish normal behavior

Send a request to the preview feature with a harmless URL controlled by the lab.

Confirm that the server fetches the URL and returns response content.

### 2. Test loopback access

In the isolated lab, change only the destination to the local training service:

```text
http://127.0.0.1:8000/
```

A successful response demonstrates that the server can access a loopback service that is not intended to be directly exposed through the application's normal interface.

### 3. Enumerate the intended lab service

The practice environment can expose a second service on another local port, for example:

```text
http://127.0.0.1:5001/
```

The analyst should test only ports and services intentionally configured as part of the local lab.

Example request using `curl` against the vulnerable local application:

```bash
curl -X POST http://127.0.0.1:8080/preview \
  -d 'url=http://127.0.0.1:5001/'
```

The returned application response should be inspected for evidence that the request reached the intended training service.

### 4. Validate the trust-boundary failure

The key finding is not simply that `127.0.0.1` responds. It is that a user-controlled URL causes the **server** to make the request.

A useful validation sequence is:

```text
Direct browser request to internal service → inaccessible
                     ↓
Application preview request → accessible
```

This demonstrates the difference between client-side reachability and server-side reachability.

## Result

The controlled practice scenario demonstrates SSRF when the preview endpoint accepts an arbitrary URL and performs the fetch from the application server.

The intended learning outcome is identification of the server-side trust-boundary violation and understanding how internal services can become reachable through an SSRF primitive.

No real internal systems, credentials, secrets, cloud metadata services, or production targets are involved.

## Key Commands / Code

Start a harmless local HTTP service:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Send a controlled request to the practice application's preview endpoint:

```bash
curl -X POST http://127.0.0.1:8080/preview \
  -d 'url=http://127.0.0.1:5001/'
```

## Lessons Learned

- URL-fetching functionality should treat destination URLs as untrusted input.
- SSRF is fundamentally a server-side trust-boundary problem.
- `127.0.0.1` and private network destinations can expose services that are not directly reachable by an external client.
- Testing should distinguish browser reachability from server-side reachability.
- SSRF testing should be performed only against systems explicitly authorized for assessment.
- URL parsing, DNS resolution, redirects, and IP-range validation all need to be considered when designing SSRF defenses.

## Remediation

A production URL-fetching feature should avoid arbitrary outbound requests where possible. If fetching is required, use a strict allowlist of permitted schemes, hosts, and destinations.

Additional controls should include:

- Resolve hostnames and validate the resulting IP addresses before connecting.
- Block loopback, link-local, private, multicast, and other non-public address ranges unless explicitly required.
- Re-check destinations after redirects and DNS resolution changes.
- Restrict supported URL schemes to the minimum required.
- Apply outbound network controls at the infrastructure layer.
- Avoid returning unnecessary upstream response content to users.

## Conclusion

This practice lab demonstrates a realistic SSRF testing workflow without interacting with real internal infrastructure. The core lesson is to identify where user-controlled data crosses a server-side network trust boundary and then validate that behavior safely inside an isolated lab.
