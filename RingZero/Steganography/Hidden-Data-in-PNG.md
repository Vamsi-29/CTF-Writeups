# Hidden Data in PNG

**Category:** Steganography  
**Technique:** PNG Metadata and LSB Analysis  
**Difficulty:** Medium

## Challenge

The challenge provides a PNG image that appears normal when opened.

The objective is to determine whether additional information has been hidden inside the image and recover the embedded data.

## Reconnaissance / Analysis

I first identified the file type and checked its basic properties.

```bash
file challenge.png
```

The output confirmed that the file was a PNG image.

I then inspected the metadata:

```bash
exiftool challenge.png
```

The metadata did not immediately reveal the hidden message, so I moved to extracting printable strings from the file.

```bash
strings challenge.png
```

This can reveal accidentally stored text, comments, or other readable data inside the image.

## Vulnerability / Technique

The challenge is based on **steganography**, where information is concealed inside another file without obviously changing its appearance.

PNG files can contain metadata and multiple types of ancillary chunks. Image data can also be manipulated at the bit level to hide information.

Because the image looked normal, the next step was to inspect it for least-significant-bit (LSB) data.

## Exploitation / Solution

A tool such as `zsteg` can be used to test common PNG bit-plane and channel combinations:

```bash
zsteg challenge.png
```

If suspicious output is identified, the relevant bit plane can be extracted for further analysis.

For example, a candidate result can be dumped with:

```bash
zsteg -E b1,rgb challenge.png > extracted.txt
```

The extracted output can then be inspected:

```bash
cat extracted.txt
```

If the data is encoded rather than directly readable, the encoding should be identified before decoding it.

## Result

The analysis demonstrates a practical workflow for investigating a PNG that may contain hidden information:

```text
PNG Image
    ↓
File Identification
    ↓
Metadata Inspection
    ↓
String Extraction
    ↓
LSB / Bit-Plane Analysis
    ↓
Hidden Data Extraction
```

## Key Commands

```bash
file challenge.png
exiftool challenge.png
strings challenge.png
zsteg challenge.png
```

## Lessons Learned

- Do not rely only on the visible appearance of an image during forensic analysis.
- File metadata should be checked before moving to more advanced techniques.
- `strings` is useful for quickly identifying embedded readable data.
- LSB analysis is a common technique for investigating PNG steganography.
- Different color channels and bit planes may contain different hidden data.

## Conclusion

This scenario demonstrates a systematic approach to PNG steganography analysis. Starting with file identification and metadata inspection before moving to bit-level analysis helps avoid unnecessary tooling and makes the investigation easier to document.

> This is an original standalone security writeup and is not presented as a solution to an official RingZer0 challenge.
