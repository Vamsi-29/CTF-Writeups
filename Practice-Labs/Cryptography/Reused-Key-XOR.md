# Cryptography Practice — Reused-Key XOR

**Category:** Cryptography  
**Technique:** Repeating-key XOR / Known-Plaintext Analysis  
**Difficulty:** Medium  
**Type:** Self-Created Practice Lab

> This is an original CTF-style practice scenario created for learning. It is not an official CTF challenge and does not represent a real solved challenge, flag, ranking, or achievement.

## Challenge / Context

A lab application encrypts several short messages using XOR with the same repeating key. The ciphertexts are available to the analyst, but the key is not.

The objective is to identify the weakness caused by key reuse and recover the plaintext of a target ciphertext in the controlled lab.

## Reconnaissance / Analysis

First, inspect the supplied ciphertexts and determine whether they were produced using the same encryption method.

For repeating-key XOR, encryption is conceptually:

```text
C[i] = P[i] XOR K[i mod key_length]
```

If the same key is reused for multiple plaintexts, XORing two ciphertexts removes the key wherever the same key bytes are aligned:

```text
C1 XOR C2 = P1 XOR P2
```

This does not immediately reveal both plaintexts, but it leaks relationships between them and can make known-plaintext or crib-dragging attacks possible.

## Vulnerability / Technique

The cryptographic weakness is **key reuse**.

A secure stream cipher or one-time-pad construction must not reuse the same keystream for independent plaintexts. Reusing a repeating XOR key creates a two-time-pad style weakness.

The main analysis steps are:

```text
Multiple ciphertexts
        ↓
Identify common encryption/key reuse
        ↓
XOR ciphertexts
        ↓
Look for probable plaintext relationships
        ↓
Recover key bytes / plaintext
        ↓
Decrypt target message
```

## Solution Steps

### 1. Convert the ciphertexts to bytes

If the challenge data is stored as hexadecimal, convert it before performing XOR operations.

```python
bytes.fromhex("...")
```

### 2. XOR two ciphertexts

A simple helper can compare two ciphertexts:

```python
def xor_bytes(a, b):
    return bytes(x ^ y for x, y in zip(a, b))

c1 = bytes.fromhex("...")
c2 = bytes.fromhex("...")

xored = xor_bytes(c1, c2)
print(xored.hex())
```

The resulting bytes represent `P1 XOR P2` rather than either plaintext directly.

### 3. Use a known plaintext fragment

Suppose analysis of the lab context gives a strong indication that a plaintext contains a predictable fragment such as:

```text
message:
```

XORing that known plaintext against the corresponding ciphertext bytes can recover the key bytes for those positions:

```python
cipher = bytes.fromhex("...")
known = b"message:"

key_part = bytes(c ^ p for c, p in zip(cipher, known))
print(key_part.hex())
```

### 4. Test the recovered key bytes

Apply the recovered key material to another ciphertext at the matching positions:

```python
def decrypt_with_key(data, key):
    return bytes(
        value ^ key[i % len(key)]
        for i, value in enumerate(data)
    )

key = bytes.fromhex("...")
target = bytes.fromhex("...")

print(decrypt_with_key(target, key))
```

If the output contains readable plaintext, continue validating the key against the other ciphertexts rather than assuming a single readable fragment proves the complete key.

## Result

In the controlled practice lab, the intended result is recovery of the target plaintext by exploiting repeated-key XOR rather than brute-forcing every possible key.

The important observation is that the encryption design leaks information because the same keystream material is reused across independent messages.

No real credentials, secrets, flags, or production data are involved in this scenario.

## Key Commands / Code

```python
# XOR two ciphertexts
xored = bytes(a ^ b for a, b in zip(c1, c2))

# Recover key bytes from known plaintext
key_part = bytes(c ^ p for c, p in zip(cipher, known))

# Repeat-key XOR decryption
plaintext = bytes(
    c ^ key[i % len(key)]
    for i, c in enumerate(ciphertext)
)
```

## Lessons Learned

- Reusing an XOR keystream across multiple messages leaks relationships between plaintexts.
- `C1 XOR C2` removes the shared key and exposes `P1 XOR P2`.
- Known plaintext can help recover corresponding key bytes.
- A readable decrypted fragment should be validated against other ciphertexts before treating the recovered key as correct.
- Stream-cipher keys and nonces must be managed according to the construction's security requirements.
- Cryptographic security depends on correct key management as well as the algorithm itself.

## Remediation

Do not reuse a fixed XOR key across independent messages.

For real applications, use a well-reviewed authenticated-encryption construction with proper nonce/IV handling and secure key management rather than implementing repeating-key XOR manually.

## Conclusion

This practice lab demonstrates how a simple encryption design can fail even when XOR itself is mathematically correct. The vulnerability comes from **reusing the same key material**, which allows relationships between ciphertexts to be exploited.
