# Self-Created Practice Lab: ELF Checksum Gate

> **Practice content — not an official CTF challenge and not a claim of a solved competition challenge.**
>
> This lab is intentionally self-created to practice static and dynamic reverse engineering of a small ELF executable. No real challenge flag, ranking, CVE, or external result is involved.

## Challenge / Context

A small Linux ELF program called `checksum_gate` accepts a candidate license string and prints either `invalid` or `accepted`.

The source code is not provided. The goal is to recover the validation logic from the binary and produce an input that satisfies it.

The exercise focuses on a practical reverse-engineering workflow rather than a particular tool:

- identify the binary and architecture
- inspect strings and symbols
- locate the input-validation function
- follow the comparison logic
- reproduce the transformation in Python
- verify the recovered logic dynamically

## Reconnaissance

Start by identifying the file type and basic ELF metadata.

```bash
file checksum_gate
checksec --file=checksum_gate
readelf -h checksum_gate
```

Then inspect readable strings and symbols:

```bash
strings -n 5 checksum_gate
nm -C checksum_gate 2>/dev/null
```

Useful observations would include strings such as:

```text
Enter license:
invalid
accepted
```

If symbols are present, inspect the likely validation function. Otherwise, use the string references to find the code that prints `accepted`.

## Static Analysis

Disassemble the executable:

```bash
objdump -d -M intel checksum_gate | less
```

The important pattern to look for is a loop that:

1. reads one byte from the supplied input,
2. applies a fixed transformation,
3. compares the transformed value with a constant table or expected value.

For example, a recovered pseudocode structure might look like:

```c
for (i = 0; i < 8; i++) {
    x = input[i];
    x ^= 0x5A;
    x = rol8(x, 3);
    if (x != expected[i])
        return 0;
}
return 1;
```

The exact constants in a real instance should be taken from the binary during analysis. They are intentionally **not presented here as a real challenge artifact or flag**.

### Why the comparison matters

The important reverse-engineering step is not guessing a password. It is identifying the transformation performed before the comparison.

Once the transformation is known, it can be inverted:

```text
x = ROL8(input ^ key, 3)

input = ROR8(x, 3) ^ key
```

That turns the problem into deterministic byte recovery.

## Dynamic Analysis

Use GDB to confirm the static interpretation.

```bash
gdb ./checksum_gate
```

Useful commands:

```gdb
break main
run
info registers
x/s $rdi
```

If the validation function is identified, set a breakpoint there:

```gdb
break validate
run
```

Step through the loop and inspect the transformed byte immediately before the comparison:

```gdb
nexti
info registers
x/8bx <address>
```

The goal is to confirm that the value observed in the debugger matches the transformation recovered from the disassembly.

## Reproducing the Transformation

A small Python helper can reproduce an 8-bit rotate-right operation:

```python
def ror8(value, count):
    value &= 0xff
    return ((value >> count) | (value << (8 - count))) & 0xff


def recover_byte(expected, key=0x5A, rotation=3):
    return ror8(expected, rotation) ^ key
```

For a real binary, populate `expected` with the comparison bytes recovered from the executable:

```python
expected = [
    # bytes recovered from the binary during analysis
]

recovered = bytes(
    recover_byte(value)
    for value in expected
)

print(recovered)
```

This is preferable to brute-forcing the whole input space because the binary exposes a reversible transformation.

## Solution Steps

1. Confirm the file is an ELF executable with `file` and `readelf`.
2. Search for `accepted`/`invalid` strings with `strings`.
3. Use `objdump` or a disassembler to locate the branch leading to the success message.
4. Identify the validation loop and record the byte transformation.
5. Determine the comparison bytes used by the loop.
6. Invert the transformation with the corresponding rotate and XOR operations.
7. Reproduce the recovered input with a short Python script.
8. Run the binary with the recovered input and verify that execution reaches the success branch.
9. Re-check the result in GDB to confirm that the comparison values match the recovered bytes.

## Result

The practice objective is achieved when the reverse-engineered input reaches the binary's existing `accepted` branch.

No competition flag or official challenge result is claimed by this writeup. The useful result is the recovered validation algorithm and the ability to reproduce its accepted input from static analysis.

## Lessons Learned

- `strings` is useful for finding behavioral anchors, but it rarely explains the complete validation logic.
- Disassembly is most useful when reduced to the relevant data flow: input → transformation → comparison → branch.
- GDB can validate a static hypothesis without requiring blind brute force.
- Simple XOR/rotation transformations are reversible when their constants and order are known.
- Reproducing a binary's logic in Python is a practical way to validate reverse-engineering conclusions.
- In real software, client-side license checks should not be treated as a strong security boundary because an attacker can inspect and modify local validation logic.

## Defensive Note

This exercise is about analysis of a deliberately simple local binary. For real licensing or authorization systems, security decisions should be enforced by a trusted server-side component where appropriate rather than relying solely on a client-side comparison routine.
