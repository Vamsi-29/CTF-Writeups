# PNG Metadata and Appended Data

> **CTF-style Practice Lab — Self-Created Scenario**  
> This is an original practice scenario created for cybersecurity learning. It is **not** an official CTF challenge and does not represent a claim of a real challenge solve, flag, ranking, CVE, or achievement.

## Challenge / Context

You are given a PNG image from an internal security-training lab. The image opens normally, but the challenge states that additional information may have been accidentally left inside the file.

The objective is to identify the hidden data using standard forensic techniques.

## Reconnaissance / Analysis

Start by identifying the file type and checking its metadata.

```bash
file training-image.png
exiftool training-image.png
```

The important observation is that the file is a valid PNG, but metadata inspection shows an unusual text field that deserves further review.

Next, inspect printable strings:

```bash
strings -a training-image.png | less
```

If a suspicious string or encoded-looking value appears, do not immediately assume it is the answer. First determine where the data exists inside the file.

## File Structure Analysis

PNG files have a defined structure consisting of chunks. A useful first check is:

```bash
pngcheck -v training-image.png
```

This can reveal unexpected chunks or data after the normal PNG structure.

Another useful check is to inspect the end of the file:

```bash
tail -c 256 training-image.png | xxd
```

If bytes exist after the expected `IEND` chunk, that is a strong indicator that additional data has been appended to the image.

## Vulnerability / Technique

The technique used in this practice scenario is **data hiding through appended content**.

The PNG itself does not need to be corrupted. An attacker can append arbitrary bytes after the logical end of an image, and many image viewers will continue to display the image normally.

The forensic objective is therefore to distinguish legitimate PNG data from bytes that were appended after `IEND`.

## Solution Steps

### 1. Confirm the PNG signature

```bash
xxd -l 32 training-image.png
```

A valid PNG starts with the standard PNG signature:

```text
89 50 4e 47 0d 0a 1a 0a
```

### 2. Locate the end of the PNG

Search for the `IEND` chunk with a hex editor or inspect the output from `pngcheck`.

```bash
pngcheck -v training-image.png
```

The `IEND` chunk marks the logical end of the PNG stream.

### 3. Extract bytes after `IEND`

Once the offset is known, copy the bytes following `IEND` into a separate file rather than modifying the original evidence.

For example:

```bash
dd if=training-image.png of=appended-data.bin bs=1 skip=<IEND_END_OFFSET>
```

Then identify the extracted content:

```bash
file appended-data.bin
strings -a appended-data.bin
```

If the extracted content is encoded, identify the encoding from its structure before decoding it.

### 4. Validate the finding

A useful forensic check is to compare the original image with a clean copy or reconstruct the PNG without the appended bytes. The image should remain visually valid after the extra content is removed.

This confirms that the suspicious content was separate from the actual image data.

## Key Commands

```bash
file training-image.png
exiftool training-image.png
strings -a training-image.png
pngcheck -v training-image.png
xxd -l 32 training-image.png
tail -c 256 training-image.png | xxd
```

## Result

The practice investigation demonstrates how an apparently normal PNG can contain additional data after its legitimate `IEND` chunk.

No real flag or secret is included in this scenario. The intended outcome is the forensic identification and extraction of the appended data.

## Lessons Learned

- Do not rely only on whether an image opens normally.
- File metadata can provide useful investigation leads.
- `strings` is useful for triage, but its output needs validation.
- PNG chunk structure can reveal content that is not obvious from the image itself.
- Always preserve the original file when performing forensic extraction.
- File-type validation should consider the complete byte stream, not only the extension.

## Defensive Perspective

For incident response, suspicious files should be analyzed as byte streams and checked for structural anomalies. Applications handling uploaded images should also validate and re-encode files where appropriate instead of trusting filenames or client-side MIME types.
