# PCAP/XOR Forensics — Agent Configuration Analysis

> **Verified solved CTF analysis.** This writeup documents a challenge the author actually solved. The original challenge name and exact flag string were not preserved in the available source notes, so neither is invented here.

## Category

Forensics / Network Traffic Analysis / Cryptography

## Context

The challenge provided a PCAP capture containing network traffic that had to be investigated for a hidden payload. The goal was to identify useful transferred data, extract the relevant artifact, analyze the encoded content, and recover the final challenge output.

## Reconnaissance / Traffic Analysis

The first step was to open the PCAP in Wireshark and identify the protocols present in the capture.

The traffic included:

- HTTP
- FTP

This immediately suggested two useful investigation paths: inspect HTTP objects/requests and examine FTP transfers for downloaded files.

### Useful Wireshark filters

```text
http
ftp
```

The HTTP and FTP streams were reviewed to identify transferred files and suspicious payloads.

## Artifact Extraction

During the traffic investigation, a file named `agent_config.json` was identified as an important transferred artifact.

The file was extracted from the capture for offline analysis rather than attempting to interpret the encoded content directly inside the packet view.

A useful workflow for similar captures is:

```text
PCAP
  ↓
Protocol identification
  ↓
HTTP / FTP stream inspection
  ↓
Transferred-file extraction
  ↓
agent_config.json
  ↓
Payload analysis
```

## Payload Analysis

The extracted configuration contained an encoded payload that did not immediately appear as plaintext.

The next step was to test whether the payload was produced using a simple reversible transformation. XOR was a strong candidate because it is common in CTF challenges and can be tested efficiently against a suspected single-byte key space.

For a single-byte XOR test, each byte can be transformed as:

```python
candidate = bytes(b ^ key for b in data)
```

A basic brute-force loop can be used to inspect printable candidates:

```python
for key in range(256):
    decoded = bytes(b ^ key for b in data)
    if all(32 <= b < 127 or b in (9, 10, 13) for b in decoded):
        print(f"key={key}: {decoded!r}")
```

## XOR Result

Testing the candidate keys produced a readable result when the XOR key was **64**.

The recovered plaintext corresponded to the challenge's final flag output. The exact flag string is intentionally not reproduced here because it is not retained in the durable source notes available for this repository update.

## Key Investigation Steps

1. Open the supplied PCAP in Wireshark.
2. Identify the protocols present in the capture.
3. Filter for HTTP and FTP traffic.
4. Inspect the relevant streams and transferred objects.
5. Extract `agent_config.json`.
6. Inspect the extracted payload offline.
7. Test simple reversible encodings/transformations.
8. Brute-force single-byte XOR keys from `0x00` through `0xff`.
9. Identify key `64` from the readable plaintext result.
10. Validate that the resulting plaintext contained the expected CTF flag output.

## Lessons Learned

- **Start with protocol identification.** Knowing whether a PCAP contains HTTP, FTP, DNS, or another protocol quickly narrows the investigation path.
- **Extract artifacts before decoding them.** Working with the transferred file separately makes repeated analysis easier and preserves the original evidence.
- **Use simple transformations early.** Single-byte XOR is inexpensive to test and is common in intentionally constructed CTF payloads.
- **Validate candidates by readability and context.** A technically possible XOR result is not enough; the decoded output should make sense in the challenge context.
- **Separate network analysis from payload analysis.** Wireshark is useful for finding the artifact, while small scripts are often better for testing transformations.

## Tools

- Wireshark
- Python
- PCAP protocol/stream analysis

## Source Integrity

This is a verified solved challenge based on the author's recorded investigation: HTTP/FTP traffic was analyzed, `agent_config.json` was extracted, the payload was tested with XOR brute force, and key `64` produced the readable flag result. No challenge name, flag value, or additional outcome has been invented.
