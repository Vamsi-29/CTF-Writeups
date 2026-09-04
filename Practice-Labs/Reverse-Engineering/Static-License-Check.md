# CTF-Style Practice Lab: Static License Check

> **Status:** Self-Created Practice Scenario
>
> This is an original practice/lab exercise created for learning reverse engineering. It is **not** an official CTF challenge, and it does not represent a claimed competition result, ranking, flag, or real-world finding.

## Challenge / Context

A Linux executable named `license_check` is provided in an isolated practice environment. The program accepts a string from standard input and prints either an acceptance or rejection message.

The objective is to understand how the executable validates the input and recover the expected input **without modifying the binary**.

No real credentials, secrets, or external systems are involved.

## Reconnaissance

Start by identifying the file type and basic metadata:

```bash
file license_check
ls -lh license_check
```

Check for readable strings:

```bash
strings -n 5 license_check | less
```

Useful observations may include:

- Error and success messages
- Function names if symbols were not stripped
- References to comparison routines
- Embedded constants

## Static Analysis

Load the binary into a reverse-engineering tool such as Ghidra and identify the `main` function.

The important questions are:

1. Where is user input read?
2. Is the input transformed before comparison?
3. What value is ultimately compared?
4. Is the comparison direct or performed through a helper function?

A typical analysis flow is:

```text
main()
  |
  +-- read input
  |
  +-- transform input
  |      |
  |      +-- character-wise operation
  |
  +-- compare transformed value
  |
  +-- success / failure branch
```

The goal is to understand the logic rather than patch the conditional jump.

## Vulnerability / Technique

This practice scenario demonstrates **static reverse engineering of client-side validation logic**.

If an executable contains the complete validation rule and expected comparison material locally, an analyst can inspect the binary and reproduce the validation logic without interacting with a remote service.

The key technique is tracing data flow from the input function to the final comparison.

## Solution Steps

### 1. Identify the input function

In Ghidra, search for references to common input functions such as `fgets`, `scanf`, or `read`.

Follow the call into the function that processes the supplied string.

### 2. Trace transformations

Inspect the instructions or decompiled code between input and comparison.

Look for simple operations such as:

```c
buffer[i] = buffer[i] ^ key;
```

or:

```c
buffer[i] = buffer[i] + offset;
```

The exact transformation should be derived from the binary during the practice exercise rather than assumed.

### 3. Identify the comparison

Locate the final comparison operation, such as `strcmp`, `memcmp`, or a loop performing byte-by-byte comparisons.

Record the data being compared and work backward through the transformation.

### 4. Reproduce the logic

Once the transformation is understood, reproduce it in a small local script. For example:

```python

def transform(data, key):
    return bytes(b ^ key for b in data)

# Use values derived from the practice binary.
# Do not hard-code secrets from real systems.
```

The purpose is to validate the reverse-engineered algorithm independently.

### 5. Validate in the isolated lab

Run the binary with the reconstructed input:

```bash
./license_check
```

Enter the derived practice input and observe the program's response.

## Result

**Expected practice result:** the reconstructed input follows the same validation path as the executable and reaches its success branch.

This result is part of the self-created lab scenario and is **not** an official CTF solve or personal competition achievement.

No flag is included because this practice exercise is intended to demonstrate the reverse-engineering methodology rather than manufacture a CTF flag.

## Key Commands

```bash
file license_check
strings -n 5 license_check
chmod +x license_check
./license_check
```

For deeper analysis:

```bash
gdb ./license_check
```

Useful GDB commands include:

```gdb
info functions
break main
run
```

## Lessons Learned

- Start reverse engineering with simple static reconnaissance before jumping into dynamic debugging.
- Trace data flow from input to the final validation decision.
- Do not confuse an application's local validation logic with secure server-side authorization.
- Reimplementing a transformation in a small script is useful for confirming understanding.
- Avoid patching the binary when the learning objective is to understand how the original logic works.
- In real applications, sensitive authorization decisions should be enforced server-side and should not rely on client-controlled validation.

## Defensive Takeaways

For software that performs security-sensitive validation:

- Do not embed reusable secrets directly in client-side binaries.
- Perform authorization decisions on a trusted server.
- Treat client-side checks as bypassable.
- Use appropriate cryptographic verification rather than custom reversible transformations.
- Minimize sensitive information exposed through compiled artifacts.
