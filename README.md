# CTF Writeups

A collection of personally solved CTF challenges used to practice practical cybersecurity skills.

## Featured Writeups

- [PCAP/XOR Forensics — Agent Configuration Analysis](Forensics/PCAP-XOR-Agent-Config-Analysis.md) — Wireshark protocol analysis, HTTP/FTP artifact extraction, `agent_config.json` investigation, single-byte XOR analysis, and payload validation.
- [picoCTF — Java Code Analysis (detailed)](picoCTF/Medium/Web%20exploitation/Java-Code-Analysis-Professional.md) — Detailed source-code review, JWT claim analysis, hard-coded signing-secret discovery, token forgery, privilege escalation, and defensive remediation.
- [picoCTF — Java Code Analysis](picoCTF/Medium/Web%20exploitation/Java-Code-Analysis.md) — Source-code review, JWT claim analysis, signing-secret discovery, token forgery, and privilege escalation.
- [picoCTF — Roboto Sans](picoCTF/Medium/Web%20exploitation/Roboto-Sans.md) — Web enumeration, `robots.txt` inspection, Base64 decoding, hidden-resource discovery, and flag retrieval.
- [picoCTF — Irish-Name-Repo 1](picoCTF/Medium/Web%20exploitation/Irish-Name-Repo-1.md) — Burp Suite request analysis, debug-parameter discovery, SQL query disclosure, SQL injection authentication bypass, and defensive remediation.
- [picoCTF — findme](picoCTF/Medium/Web%20exploitation/Findme.md) — Burp Suite traffic interception, HTTP redirect analysis, Base64 fragment reconstruction, and flag decoding.
- [picoCTF — JAuth](picoCTF/Medium/Web%20exploitation/JAuth.md) — JWT cookie analysis, weak signature validation, `alg: none` manipulation, role modification, and privilege escalation.
- [picoCTF — Irish-Name-Repo 3](picoCTF/Web%20Exploitation/Medium/Irish-Name-Repo%203.md) — UNION-based SQL injection, database enumeration, and credential extraction.
- [picoCTF — Forbidden Paths](picoCTF/Web%20Exploitation/Medium/Forbidden-Paths.md) — Path traversal through a file-reading function to bypass absolute-path filtering and retrieve the flag.
- [picoCTF — Crack the Gate 1](picoCTF/Web%20Exploitation/Easy/Crack-the-Gate-1.md) — HTML source inspection, ROT13 decoding, hidden HTTP-header discovery, and authentication-flow manipulation.
- [picoCTF — Inspect HTML](picoCTF/Easy/Web%20exploitation/Inspect-HTML.md) — HTML source inspection and information-disclosure analysis.
- [picoCTF — Log Hunt](picoCTF/Easy/General%20skills/Log-Hunt.md) — Server-log filtering, `INFO FLAGPART` identification, fragment collection, duplicate handling, and flag reconstruction.
- [picoCTF — Lets Warm Up](picoCTF/Easy/General%20skills/Lets-Warm-Up.md) — Hexadecimal-to-ASCII conversion, command-line verification, and CTF flag-format handling.

## Practice Labs

The following section contains **self-created CTF-style practice scenarios** for documenting cybersecurity concepts. These are intentionally separate from verified solved CTF writeups and are not presented as official challenge solutions or personal achievements.

- [Web Exploitation — IDOR Authorization Bypass](Practice-Labs/Web/IDOR-Authorization-Bypass.md) — Object-level authorization testing, authenticated API request analysis, identifier manipulation, horizontal access-control validation, and remediation.
- [Web Exploitation — SSRF Local Service Discovery](Practice-Labs/Web/SSRF-Local-Service-Discovery.md) — Server-side URL fetching, loopback reachability, trust-boundary validation, controlled SSRF testing, and remediation.
- [Web Exploitation — XXE Local File Read](Practice-Labs/Web/XXE-Local-File-Read.md) — XML parser analysis, external entity resolution, controlled local-file disclosure testing, and remediation.
- [Web Exploitation — JWT Algorithm Confusion](Practice-Labs/Web/JWT-Algorithm-Confusion.md) — JWT header analysis, algorithm/key confusion, controlled token manipulation, authorization impact, and secure verification practices.
- [Cryptography — Repeating-Key XOR Analysis](Practice-Labs/Cryptography/Repeating-Key-XOR-Analysis.md) — XOR properties, known-plaintext reasoning, repeating-key recovery, decryption, and validation.
- [Cryptography — AES-CBC Bit Flipping](Practice-Labs/Cryptography/AES-CBC-Bit-Flipping.md) — CBC block analysis, ciphertext manipulation, XOR mask calculation, authorization-state tampering, and authenticated-encryption remediation.
- [Forensics — PNG LSB Steganography](Practice-Labs/Forensics/PNG-LSB-Steganography.md) — PNG metadata triage, RGB least-significant-bit extraction, byte reconstruction, hidden-data validation, and evidence hashing.
- [Forensics — Windows Event Log Timeline Analysis](Practice-Labs/Forensics/Windows-Event-Log-Timeline.md) — Windows authentication, process-creation, and PowerShell telemetry correlation for timeline reconstruction.
- [Forensics — PDF Embedded Object Analysis](Practice-Labs/Forensics/PDF-Embedded-Object-Analysis.md) — PDF object enumeration, embedded-file identification, safe stream extraction, file-type validation, hashing, and forensic handling.
- [Network Security — TCP Service Enumeration and Banner Analysis](Practice-Labs/Networking/TCP-Service-Enumeration.md) — TCP service discovery, version detection, HTTP response validation, banner analysis, evidence collection, and investigation planning.
- [Network Security — DNS Exfiltration PCAP](Practice-Labs/Network-Security/DNS-Exfiltration-PCAP.md) — DNS traffic analysis, encoded-query detection, fragment reconstruction, PCAP investigation, and defensive monitoring.
- [Linux Privilege Escalation — SUID Misconfiguration](RingZero/Practice-Labs/Linux-Privilege-Escalation/SUID-Misconfiguration.md) — SUID enumeration, binary analysis, unsafe command lookup, PATH manipulation, exploitation flow, and remediation.
- [Reverse Engineering — Static License Check](Practice-Labs/Reverse-Engineering/Static-License-Check.md) — Static binary analysis, input-to-comparison data flow, transformation recovery, GDB reconnaissance, and secure validation design.
- [Reverse Engineering — ELF Checksum Gate](Practice-Labs/Reverse-Engineering/ELF-Checksum-Gate.md) — ELF reconnaissance, disassembly, input-validation data flow, reversible XOR/rotation analysis, GDB validation, and Python reproduction.

## Coverage

- 🌐 Web exploitation
- 🔐 Cryptography
- 🕵️ Forensics
- 🌐 Network security
- 🧩 Miscellaneous security challenges
- 🐧 Linux privilege escalation
- ⚙️ Reverse engineering

## Platforms / Events

- CyberThreya
- picoCTF
- RingZer0 CTF
- Hack The Box

## Writeup format

Where applicable, writeups document:

1. Challenge objective
2. Reconnaissance and observations
3. Vulnerability or attack technique
4. Exploitation / solution approach
5. Flag or final result
6. Key lesson learned

The purpose is to demonstrate **methodology and problem-solving**, not simply record completed challenges.

> CTF content is performed in authorized challenge environments. Practice-lab content is clearly separated from verified solved challenges.
