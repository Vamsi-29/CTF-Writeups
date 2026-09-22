# picoCTF — Lets Warm Up

## Challenge / Context

- **Platform:** picoCTF
- **Category:** General Skills
- **Difficulty:** Easy
- **Challenge:** Lets Warm Up

The challenge provided a hexadecimal value, `0x70`, and asked what it represents in ASCII.

## Objective

Convert the hexadecimal value to its ASCII representation and submit the result using the challenge's required flag format.

## Analysis

The supplied value was:

```text
0x70
```

The hexadecimal byte `0x70` corresponds to the ASCII character `p`.

The original solve notes record the same conversion and the required `picoCTF{...}` submission format. fileciteturn9file0L2-L2

## Solution

### Step 1 — Identify the encoding

The `0x` prefix indicates hexadecimal notation. The task therefore requires a hexadecimal-to-ASCII conversion.

### Step 2 — Convert `0x70`

This can be verified directly from a shell:

```bash
printf '\\x70\n'
```

Output:

```text
p
```

The same conversion can also be performed with a decoder such as Burp Suite Decoder, which was the approach documented in the original solve notes. fileciteturn9file0L2-L2

### Step 3 — Apply the flag format

The decoded character is placed inside the challenge's flag wrapper:

```text
picoCTF{p}
```

## Result

The challenge was solved with:

```text
picoCTF{p}
```

This result is preserved in the user's original solve notes rather than being reconstructed as a new or assumed result. fileciteturn9file0L2-L2

## Key Takeaways

- Recognize common encoding prefixes such as `0x` for hexadecimal.
- Understand that hexadecimal bytes can map directly to ASCII characters.
- Verify simple transformations with command-line tools when possible.
- Preserve the exact submission format required by a CTF platform.

## Lessons Learned

Even very small CTF challenges reinforce useful fundamentals. Before reaching for complex tooling, identify the data representation and perform the simplest valid transformation first. Hexadecimal and ASCII conversion is a basic skill that appears frequently in web, forensics, networking, and binary-analysis challenges.
