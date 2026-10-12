# CTF-Style Practice Lab — ELF String Obfuscation Analysis

> **Practice content:** This is a self-created lab scenario for learning reverse-engineering methodology. It is not an official CTF challenge and does not represent a real binary, flag, ranking, CVE, or personal achievement.

## Challenge / Context

A small Linux ELF program asks the user for a validation string. The expected value is not visible in normal printable strings because the program stores it in a simple transformed form.

The objective is to analyze the binary statically, identify the input-validation routine, recover the transformation, reproduce it in Python, and verify the result locally.

The binary is a disposable training sample. No credentials or real software are involved.

## Reconnaissance / Analysis

Start with basic file identification:

    file ./validator
    checksec --file=./validator
    sha256sum ./validator

Then inspect printable strings:

    strings -n 5 ./validator

If the expected validation string is not visible, inspect ELF metadata:

    readelf -h ./validator
    readelf -S ./validator
    readelf -s ./validator

The goal is to locate the function that reads the supplied input and compares it with an internally generated value.

## Technique

The practice binary uses a simple reversible byte transformation rather than storing the expected string directly.

Example transformation:

    transformed = rotate_left(original XOR key, 3)

This is intentionally a toy transformation for learning and is not a real cryptographic construction.

## Solution Steps

### 1. Find the input-handling code

    objdump -d -M intel ./validator | less
    objdump -d -M intel ./validator | grep -E 'strcmp|strncmp|fgets|scanf|read'

Calls associated with input or string comparison provide useful anchors for locating the validation path.

### 2. Inspect the comparison routine

    objdump -d -M intel ./validator | sed -n '120,220p'

Look for a sequence that reads user input, iterates over bytes, applies XOR or rotation instructions, and compares the transformed value with bytes stored in the binary.

Useful x86 instructions to recognize include xor, rol, ror, cmp, movzx, and mov.

### 3. Reverse the transformation

Suppose static analysis shows:

    byte -> XOR with 0x23 -> rotate left by 3

Reverse the operations in the opposite order:

    encoded -> rotate right by 3 -> XOR with 0x23

The reverse transformation should be derived from observed instructions rather than guessed.

### 4. Reproduce it in Python

    def ror8(value, shift):
        shift %= 8
        return ((value >> shift) | (value << (8 - shift))) & 0xff

    encoded = bytes([
        # Lab-specific transformed bytes go here.
    ])

    decoded = bytes(ror8(b, 3) ^ 0x23 for b in encoded)
    print(decoded)

The byte sequence is intentionally left as a lab-specific placeholder rather than embedding a fabricated flag or credential.

### 5. Confirm with the binary

    ./validator

A successful validation should match the behavior predicted by static analysis.

## GDB Validation

Use GDB to confirm the data flow without modifying the executable:

    gdb ./validator

Useful commands:

    break main
    run
    disassemble
    info registers
    x/16bx ADDRESS

Set a breakpoint around the comparison instruction and inspect the relevant registers and memory.

The expected data flow is:

    User input -> Transformation -> Expected byte sequence -> Comparison

## Result

**Practice result:** the analyst can recover a simple reversible transformation from the ELF disassembly and reproduce the validation logic independently in Python.

No real secret, credential, flag, or third-party software is claimed.

## Defensive Notes

Simple client-side transformations should not be treated as secure secret storage.

- Do not embed authentication secrets directly in client-side binaries.
- Do not rely on reversible obfuscation for access control.
- Perform authorization decisions on a trusted server.
- Use established cryptographic primitives when confidentiality or integrity is required.
- Treat compiled binaries as inspectable by the user.

## Lessons Learned

- Start reverse engineering with basic ELF triage before dynamic analysis.
- Printable strings are useful, but missing readable strings do not mean data is protected.
- Disassembly reveals the actual transformation applied to input.
- Reversible operations should be undone in reverse order.
- Python is useful for reproducing small byte-level algorithms.
- GDB provides a practical way to validate static-analysis conclusions.

## Practice Classification

This document is intentionally stored under Practice-Labs/ to distinguish it from verified CTF writeups. It is self-created training content for practicing ELF reverse-engineering methodology in an authorized lab.