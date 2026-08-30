# RingZero Practice — Steganography Image Analysis

> **Practice writeup:** This is a self-created CTF-style lab scenario. It is not an official RingZer0 challenge and is not presented as a challenge previously solved by the author.

## Challenge / Context

The lab provides an image that appears normal at first glance. The objective is to determine whether information has been hidden inside the image and recover the embedded message.

## Reconnaissance / Analysis

I started by checking the file type and basic metadata rather than immediately modifying the image.

```bash
file challenge.png
exiftool challenge.png
```

The file was a valid PNG, but the metadata and visible image did not explain the presence of the challenge data.

I then checked whether the PNG contained unusual strings:

```bash
strings challenge.png | less
```

If the expected message is not visible in the output, the next step is to investigate common PNG-based hiding techniques.

## Vulnerability / Technique

The technique is **steganography**: information is concealed inside another file so that its presence is not obvious from normal inspection.

For an image challenge, useful checks include:

- File signature and format validation
- Metadata
- Embedded text
- Additional appended data
- Image dimensions and color channels
- Least-significant-bit (LSB) encoding

## Solution Approach

I first compared the file size and metadata with the expected properties of the image. I then used a steganography-analysis tool in the controlled lab to test common embedded-data techniques.

A typical first-pass workflow is:

```bash
file challenge.png
exiftool challenge.png
strings challenge.png
```

For a PNG suspected of containing LSB data, a tool such as `zsteg` can be used to enumerate likely bit-plane encodings:

```bash
zsteg challenge.png
```

The useful finding in this practice scenario is an embedded text stream in one of the image's low-order bit planes. Extracting that stream reveals the challenge message.

## Result

The practice scenario demonstrates that a visually normal image can contain additional information that is not apparent through ordinary viewing.

Investigation flow:

```text
Image
 ↓
File / metadata inspection
 ↓
String extraction
 ↓
Steganography analysis
 ↓
Bit-plane identification
 ↓
Hidden message recovered
```

## Lessons Learned

- Start with basic file and metadata analysis before using specialized tooling.
- A valid image can contain data outside the visible pixels or normal metadata.
- `strings` is useful for quick triage, but absence of readable text does not rule out steganography.
- Automated steganography tools can quickly identify likely bit-plane encodings.
- Always preserve the original evidence when performing forensic-style analysis.

## Key Takeaway

Good CTF forensics is usually systematic: **identify the file → inspect metadata → search for obvious artifacts → test likely hiding techniques → validate the recovered data**.
