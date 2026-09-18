# PDF Embedded Object Analysis

> **CTF-style Practice Lab — Self-Created Scenario**  
> This is an original practice scenario created for cybersecurity learning. It is **not** an official CTF challenge and does not represent a claim of a real challenge solve, flag, ranking, CVE, or achievement.

## Challenge / Context

You are given a PDF from an authorized security-training lab. The document opens normally and appears to contain only a short report, but the challenge states that an additional object may be embedded inside the PDF.

The objective is to determine whether the PDF contains embedded content and safely extract it for analysis.

## Reconnaissance / Analysis

Start by identifying the file and reviewing its basic metadata.

```bash
file training-report.pdf
pdfinfo training-report.pdf
```

Next, inspect the PDF for readable strings and object references:

```bash
strings -a training-report.pdf | less
```

For structured PDF inspection, `pdf-parser.py` can be used to enumerate objects and identify suspicious object types:

```bash
pdf-parser.py training-report.pdf
```

The important observation in this practice scenario is that the PDF contains an object associated with an embedded file rather than only normal page-content objects.

## Vulnerability / Technique

The technique is **embedded-file discovery in a PDF**.

PDF documents can contain attachments and other embedded objects while still appearing to be ordinary documents to the user. From a forensic perspective, the visible document is therefore not necessarily the complete content of the file.

The investigation focuses on identifying the embedded object, extracting it without executing it, and validating its file type.

## Solution Steps

### 1. Verify the document type

```bash
file training-report.pdf
pdfinfo training-report.pdf
```

Confirm that the file is actually a PDF and record useful metadata before extraction.

### 2. Enumerate PDF objects

Use a PDF parser to inspect the object structure:

```bash
pdf-parser.py training-report.pdf
```

Look for objects related to embedded files, file specifications, or streams. Typical indicators include `/EmbeddedFile` and `/Filespec` entries.

### 3. Locate the embedded object

Search the parser output for embedded-file indicators:

```bash
pdf-parser.py training-report.pdf | grep -Ei 'EmbeddedFile|Filespec'
```

Once the relevant object number is identified, inspect that object:

```bash
pdf-parser.py training-report.pdf -o <OBJECT_ID>
```

The goal is to understand the object relationship and locate the embedded stream. Do not execute extracted content during this stage.

### 4. Extract the embedded stream

After identifying the relevant stream object, extract it to a separate file using the parser's stream-extraction functionality or an equivalent PDF forensic tool.

For example:

```bash
pdf-parser.py training-report.pdf -o <STREAM_OBJECT_ID> -d extracted-object.bin
```

The exact object ID is determined from the analysis of the supplied PDF; it should never be guessed.

### 5. Identify the extracted data

Treat the extracted object as untrusted evidence:

```bash
file extracted-object.bin
sha256sum extracted-object.bin
strings -a extracted-object.bin | head
```

If the extracted data has a recognizable file signature, rename it only after confirming its type. Do not execute an unknown binary or script simply because it was embedded in a PDF.

### 6. Preserve forensic integrity

Keep the original PDF unchanged and perform extraction into a separate working directory.

A useful workflow is:

```text
Original PDF
     ↓
Object Enumeration
     ↓
Embedded-File Identification
     ↓
Stream Extraction
     ↓
File-Type Validation
     ↓
Static Analysis
```

## Key Commands

```bash
file training-report.pdf
pdfinfo training-report.pdf
strings -a training-report.pdf
pdf-parser.py training-report.pdf
pdf-parser.py training-report.pdf | grep -Ei 'EmbeddedFile|Filespec'
pdf-parser.py training-report.pdf -o <OBJECT_ID>
pdf-parser.py training-report.pdf -o <STREAM_OBJECT_ID> -d extracted-object.bin
file extracted-object.bin
sha256sum extracted-object.bin
```

## Result

The practice investigation demonstrates how to identify an embedded object inside an otherwise normal PDF, extract it as separate evidence, and perform safe static analysis on the extracted content.

No real flag, secret, credential, or malicious payload is included in this scenario. The intended outcome is successful identification and forensic extraction of the embedded object.

## Lessons Learned

- A document that opens normally can still contain additional embedded content.
- PDF object structure is useful during document forensics.
- `/EmbeddedFile` and `/Filespec` are useful indicators when triaging PDF attachments.
- Extract suspicious content before analyzing it rather than opening it directly from the document.
- `file` and cryptographic hashes provide useful validation and evidence-handling information.
- Never execute unknown extracted files during initial forensic analysis.

## Defensive Perspective

Organizations should treat externally supplied PDFs as untrusted input and inspect embedded objects when investigating suspicious documents. Email and document-security controls can restrict or flag embedded attachments, while sandboxing and static analysis can reduce the risk of executing malicious content.
