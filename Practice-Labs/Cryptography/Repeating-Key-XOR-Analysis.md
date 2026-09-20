# Practice Lab — Repeating-Key XOR Analysis

> **Practice content:** This is a self-created CTF-style laboratory scenario. It is not an official CTF challenge, not a real-world incident, and not a claim of a completed competition challenge.

**Category:** Cryptography  
**Difficulty:** Medium  
**Techniques:** XOR properties, known-plaintext reasoning, key-length analysis, repeating-key recovery

## Challenge / Context

A fictional application stores a short configuration message as hexadecimal ciphertext. The developer attempted to protect the message by XORing it with a short key and repeating that key across the plaintext.

The lab objective is to recover the plaintext by analyzing the XOR construction. No real credential, secret, flag, or production data is used.

## Reconnaissance / Analysis

The supplied ciphertext is hexadecimal, so the first step is to convert it into bytes.

The important observation is that XOR with a repeating key creates a repeating structure:

```text
P[i] XOR K[i mod key_length] = C[i]
```

If the key is shorter than the message, bytes at positions separated by the key length are encrypted with the same key byte.

For this practice lab, the ciphertext is intentionally constructed from a known-format configuration message and a short repeating key. That makes it possible to demonstrate key recovery without relying on a fabricated real-world vulnerability.

## Vulnerability / Technique

The weakness is not XOR itself. The problem is **reusing a short key cyclically** across a longer plaintext.

With a repeating-key XOR scheme, the same key byte encrypts multiple plaintext positions. If some plaintext structure is predictable, the corresponding key bytes can be recovered:

```text
key_byte = ciphertext_byte XOR known_plaintext_byte
```

Once enough key bytes are recovered, the complete ciphertext can be decrypted.

## Solution Steps

### 1. Convert hexadecimal to bytes

A small Python helper can decode the ciphertext representation:

```python
ciphertext = bytes.fromhex(hex_data)
```

### 2. Identify likely plaintext structure

Configuration-style data commonly contains predictable syntax such as:

```text
config=
```

or separators such as `=` and `;`.

For a controlled CTF-style exercise, a known prefix can be used to demonstrate the recovery technique.

### 3. Recover key bytes

For each known plaintext byte:

```python
key_byte = ciphertext[i] ^ known_plaintext[i]
```

Collecting bytes at the corresponding repeating positions reconstructs the short XOR key.

### 4. Decrypt with the recovered repeating key

```python
def xor_decrypt(ciphertext, key):
    return bytes(
        byte ^ key[i % len(key)]
        for i, byte in enumerate(ciphertext)
    )
```

The recovered key is then applied cyclically across the ciphertext.

### 5. Validate the plaintext

The result should be checked for expected structure rather than accepting arbitrary printable output. For example:

```python
plaintext = xor_decrypt(ciphertext, key)
print(plaintext.decode("utf-8"))
```

A coherent configuration-style message confirms that the recovered key and plaintext are consistent with the construction used by the lab.

## Result

The repeating-key XOR construction can be reversed by exploiting predictable plaintext structure and the XOR operation's reversible property.

The lab intentionally does **not** define or claim an invented competition flag or real-world result. The useful outcome is demonstrating the complete cryptanalysis workflow:

```text
Hex Ciphertext
      ↓
Hex Decode
      ↓
Identify Repeating-Key XOR
      ↓
Recognize Known Plaintext Structure
      ↓
Recover Key Bytes
      ↓
Reconstruct Repeating Key
      ↓
Decrypt
      ↓
Validate Plaintext
```

## Key Lessons

- XOR is reversible because `A XOR B XOR B = A`.
- A short repeating key leaks structure when reused across a longer plaintext.
- Known plaintext can reveal corresponding XOR key bytes.
- Cryptanalysis should validate recovered plaintext instead of relying only on printable output.
- Strong encryption requires a construction appropriate for the threat model; a repeated short XOR key is not secure encryption.

## Defensive Takeaway

Do not implement application encryption with ad-hoc repeating-key XOR. Use a well-reviewed authenticated-encryption construction with proper key generation, nonce/IV handling, and secure key storage.

> **Scope:** Self-created practice material for authorized laboratory learning. No real credentials, secrets, production targets, CVEs, rankings, or official CTF results are involved.
