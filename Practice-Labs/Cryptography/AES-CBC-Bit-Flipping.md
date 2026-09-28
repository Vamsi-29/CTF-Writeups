# CTF-Style Practice Lab — AES-CBC Bit Flipping

> **Practice content only:** This is a self-created CTF-style lab scenario for learning. It is not an official CTF challenge, and it does not claim a real challenge solve, flag, ranking, CVE, or competition result.

## Challenge / Context

A practice web service issues an encrypted session value after login. The application uses AES-CBC with a fixed-format plaintext:

```text
user=vamsi;role=user;expires=...
```

The server decrypts the cookie and trusts the `role` field. The goal is to understand whether an attacker can modify the encrypted value without knowing the AES key.

The lab is intentionally designed to demonstrate **CBC bit-flipping**. No real credentials or production data are involved.

## Reconnaissance / Analysis

Start by inspecting the response after authentication and identify the session cookie:

```http
Set-Cookie: session=<base64-encoded-ciphertext>; HttpOnly
```

The cookie length is consistent with block-oriented encryption. After Base64 decoding, the ciphertext length is a multiple of the AES block size (16 bytes).

The important observation is that CBC decryption has the form:

```text
P[i] = D(K, C[i]) XOR C[i-1]
```

Therefore, changing selected bits in `C[i-1]` changes the corresponding bits in the decrypted plaintext block `P[i]`.

The attacker does not need to recover the AES key. The challenge is to align the target bytes and calculate the required XOR mask.

## Vulnerability / Technique

The weakness is not AES itself. AES-CBC remains cryptographically strong when used correctly.

The vulnerable design combines:

1. CBC encryption without authenticated integrity protection.
2. A predictable plaintext format.
3. Server-side trust of attacker-controlled decrypted fields.

Because there is no MAC or AEAD authentication step, the server cannot detect that the ciphertext was modified.

## Solution Steps

### 1. Capture the original cookie

Save the Base64-decoded ciphertext as binary data:

```bash
printf '%s' '<COOKIE_VALUE>' | base64 -d > session.bin
```

Do this only against the intentionally created practice service.

### 2. Determine the target block

The attacker first determines which ciphertext block controls the plaintext containing the target field.

For example, if the target plaintext block contains:

```text
role=user;
```

and the desired value is:

```text
role=admin;
```

the differing bytes are calculated with XOR.

The transformation for one byte is:

```text
modified_previous_byte = original_previous_byte ^ ord('u') ^ ord('a')
```

For several bytes, apply the same calculation independently to each position.

### 3. Calculate the ciphertext modification

A small Python helper can calculate the required XOR mask:

```python
old = b"user"
new = b"admin"

for before, after in zip(old, new):
    print(f"0x{before ^ after:02x}")
```

The replacement must have the same byte length as the original plaintext at the modified positions. If the fields are different lengths, a different lab layout or padding strategy is required; CBC bit flipping does not provide arbitrary plaintext insertion.

### 4. Modify the preceding ciphertext block

Conceptually:

```python
modified = bytearray(ciphertext)

# Example offsets only; determine the real offsets from the lab ciphertext.
for offset, before, after in changes:
    modified[offset] ^= before ^ after

print(base64.b64encode(modified).decode())
```

The offsets above are deliberately not fixed challenge data. They must be derived from the practice instance being tested.

### 5. Replay the modified session

Send the modified cookie to the practice endpoint:

```http
GET /profile HTTP/1.1
Host: lab.example
Cookie: session=<MODIFIED_COOKIE>
```

The server decrypts the modified ciphertext. The targeted plaintext bytes change while the rest of the affected block may become corrupted.

### 6. Validate the result

A successful lab result is observed when the application interprets the modified plaintext according to the altered role value and grants the corresponding simulated authorization path.

The important validation is the authorization decision, not simply whether the ciphertext remains syntactically valid.

## Result

**Practice-lab outcome:** the CBC construction allows controlled modification of selected plaintext bytes without knowledge of the encryption key, demonstrating why encryption without integrity protection is unsafe for security-sensitive session state.

No real flag or external system is involved in this exercise.

## Why the Attack Works

CBC provides confidentiality, but unauthenticated CBC does not provide integrity.

For a target block:

```text
P[i] = D(K, C[i]) XOR C[i-1]
```

Changing `C[i-1]` therefore changes `P[i]` predictably:

```text
P'[i] = P[i] XOR Δ
```

The attacker controls `Δ` without learning `D(K, C[i])` or the AES key.

The trade-off is that modifying one ciphertext block can also cause the preceding plaintext block to become invalid or corrupted. A practical attack therefore requires careful block alignment.

## Defensive Remediation

Do not rely on unauthenticated AES-CBC for authorization-bearing cookies or other security-sensitive state.

Prefer an authenticated-encryption construction such as AES-GCM or ChaCha20-Poly1305. If CBC must be retained for compatibility, use a secure encrypt-then-MAC construction with strict verification before decryption results are trusted.

Additional controls:

- Keep authorization decisions server-side where possible.
- Never treat encrypted client-side data as trustworthy merely because it decrypts.
- Reject modified or unauthenticated ciphertext.
- Use secure random nonces/IVs according to the selected construction.
- Separate authentication from authorization and validate the user's privileges on the server.

## Lessons Learned

- Encryption and integrity are separate security properties.
- AES-CBC without authentication is vulnerable to ciphertext manipulation.
- Predictable plaintext makes CBC bit-flipping attacks easier to construct.
- Authorization data should not be trusted merely because it is encrypted.
- When analyzing encrypted cookies, first identify the mode, block size, plaintext structure, and presence or absence of authentication.
- For real assessments, perform this testing only against systems where you have explicit authorization.
