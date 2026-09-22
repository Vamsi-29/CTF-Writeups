# Self-Created Practice Lab — PNG LSB Steganography

> **Status:** Self-created CTF-style practice content. This is not an official CTF challenge and is not a claim of a solved competition challenge.

## Category

Forensics / Steganography

## Challenge / Context

A PNG image is provided as a forensic artifact. The image opens normally and does not show an obvious message, but the task is to determine whether information has been hidden in the pixel data.

The goal is to identify the hiding technique, extract the hidden message, and document the analysis without modifying the original evidence.

## Reconnaissance / Analysis

Start by identifying the file type and checking basic metadata.

```bash
file sample.png
exiftool sample.png
pngcheck -v sample.png
sha256sum sample.png
```

The file is a valid PNG, so the next step is to inspect the pixel channels rather than treating it as a damaged or renamed archive.

A useful first check is whether the image has an unusual color mode or dimensions:

```python
from PIL import Image

img = Image.open("sample.png")
print(img.mode)
print(img.size)
```

For an RGB/RGBA image, each pixel contains channel values that can carry more information than is visible to the eye. A common steganography technique is to store message bits in the least significant bit (LSB) of each color channel.

## Technique

### LSB extraction

The least significant bit changes a channel value by only 1, which normally produces no obvious visual difference. For example:

```text
10010110 -> LSB = 0
10010111 -> LSB = 1
```

Extract the LSB from each RGB channel and group the bits into bytes.

```python
from PIL import Image

img = Image.open("sample.png").convert("RGB")

bits = []

for r, g, b in img.getdata():
    bits.extend([r & 1, g & 1, b & 1])

data = bytearray()

for i in range(0, len(bits) - 7, 8):
    value = 0
    for bit in bits[i:i + 8]:
        value = (value << 1) | bit
    data.append(value)

print(data[:200])
```

If the output begins with readable ASCII, the extraction hypothesis is supported. If it is mostly random data, test the extraction assumptions systematically rather than treating every byte sequence as a valid message.

## Validation

A practical validation step is to check the printable-character ratio of the extracted bytes:

```python
import string

sample = bytes(data)
printable = sum(chr(b) in string.printable for b in sample)
ratio = printable / max(len(sample), 1)
print(f"Printable ratio: {ratio:.2%}")
```

A readable region can then be located with:

```bash
strings -n 6 extracted.bin
```

For a real investigation, preserve the original PNG and write extracted material to a separate file. Hash both artifacts so the analysis remains reproducible:

```bash
sha256sum sample.png extracted.bin
```

## Solution Steps

1. Confirm that the artifact is a PNG with `file` and `pngcheck`.
2. Record metadata and the SHA-256 hash before analysis.
3. Open the image with Pillow and confirm its color mode.
4. Extract the least significant bit from each RGB channel.
5. Reconstruct bytes from the extracted bit stream.
6. Inspect the resulting bytes for printable text and structural markers.
7. Save the extracted data separately instead of altering the source image.
8. Hash the extracted artifact for repeatability.

No real CTF flag is included in this practice scenario. The intended result is the recovery and validation of the hidden message from the supplied practice artifact.

## Key Commands / Code

```bash
file sample.png
exiftool sample.png
pngcheck -v sample.png
sha256sum sample.png
strings -n 6 extracted.bin
```

The core extraction logic is based on:

```python
bit = channel_value & 1
```

followed by grouping eight extracted bits into one byte.

## Result

The practice workflow demonstrates how a visually normal PNG can be investigated for LSB-based hidden data. The important outcome is the repeatable extraction methodology rather than an invented flag or competition result.

## Lessons Learned

- A valid image can contain information that is not visible during normal viewing.
- Metadata checks should be performed before deeper pixel-level analysis.
- LSB extraction is simple but depends on the correct channel order and bit grouping.
- Evidence should be hashed before and after extraction to maintain traceability.
- Extracted data should be written to separate files so the original artifact remains unchanged.
- In a real CTF, tools such as `zsteg` can accelerate PNG/BMP steganography triage, but manual extraction is useful for understanding what the tool is detecting.

## Practice Lab Notice

This document is intentionally self-created practice content for the `CTF-Writeups` repository. It is clearly separated from verified solved CTF writeups and must not be interpreted as an official challenge solution, ranking, vulnerability disclosure, or personal competition achievement.
