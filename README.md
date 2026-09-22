# CTF Writeups

A collection of personally solved CTF challenges used to practice practical cybersecurity skills.

## Featured Writeups

- [CyberThreya — Drupal 7 Login Page](CyberThreya/Web/Drupal-7-Login-Page.md) — Nmap reconnaissance, Drupal version fingerprinting, vulnerability research, Metasploit-based exploitation, and post-exploitation enumeration.
- [CyberThreya — Kira Privilege Escalation](Cyberthreya/Privilege-Escalation-Kira.md) — Web enumeration, exposed-file discovery, SSH credential reuse, Base64 decoding, local enumeration, and privilege escalation.
- [CyberThreya — Web Recon, FTP Access, and SSH Credential Reuse](Cyberthreya/Web-Recon-FTP-SSH-Credential-Reuse.md) — Source-code reconnaissance, exposed-service analysis, credential discovery, FTP enumeration, credential reuse, and SSH-based flag retrieval.
- [CyberThreya — Web Source Disclosure and FTP/SSH Credential Reuse](Cyberthreya/Web-Source-Disclosure-FTP-SSH-Credential-Reuse.md) — HTML source inspection, exposed-service analysis, credential discovery, credential reuse, and SSH-based flag retrieval.
- [CyberThreya — 192.168.1.39 FTP/SSH Credential Reuse](Cyberthreya/Web/FTP-SSH-Credential-Reuse-192.168.1.39.md) — Web source inspection, username disclosure, FTP enumeration, credential discovery, permission analysis, SSH credential reuse, and flag retrieval.
- [picoCTF — Crack the Gate 1](picoCTF/Web%20Exploitation/Easy/Crack-the-Gate-1.md) — HTML source inspection, ROT13 decoding, hidden HTTP-header discovery, and authentication-flow manipulation.
- [picoCTF — Inspect HTML](picoCTF/Web%20Exploitation/Easy/Inspect-HTML.md) — HTML source inspection and information-disclosure analysis.
- [picoCTF — Log Hunt](picoCTF/Easy/General%20skills/Log-Hunt.md) — Server-log filtering, `INFO FLAGPART` identification, fragment collection, duplicate handling, and flag reconstruction.
- [picoCTF — Lets Warm Up](picoCTF/Easy/General%20skills/Lets-Warm-Up.md) — Hexadecimal-to-ASCII conversion, command-line verification, and CTF flag-format handling.
- [picoCTF — Irish-Name-Repo 3](picoCTF/Web%20Exploitation/Medium/Irish-Name-Repo%203.md) — UNION-based SQL injection, database enumeration, and credential extraction.
- [picoCTF — Forbidden Paths](picoCTF/Web%20Exploitation/Medium/Forbidden-Paths.md) — Path traversal through a file-reading function to bypass absolute-path filtering and retrieve the flag.
- [picoCTF — JAuth](picoCTF/Medium/Web%20exploitation/JAuth.md) — JWT cookie analysis, weak signature validation, `alg: none` manipulation, role modification, and privilege escalation.
- [picoCTF — Java Code Analysis](picoCTF/Medium/Web%20exploitation/Java-Code-Analysis.md) — Source-code review, JWT claim analysis, signing-secret discovery, token forgery, and privilege escalation.

## Practice Labs

The following section contains **self-created CTF-style practice scenarios** for documenting cybersecurity concepts. These are intentionally separate from verified solved CTF writeups and are not presented as official challenge solutions or personal achievements.

- [Web Exploitation — IDOR Authorization Bypass](Practice-Labs/Web/IDOR-Authorization-Bypass.md) — Object-level authorization testing, authenticated API request analysis, identifier manipulation, horizontal access-control validation, and remediation.
- [Web Exploitation — SSRF Local Service Discovery](Practice-Labs/Web/SSRF-Local-Service-Discovery.md) — Server-side URL fetching, loopback reachability, trust-boundary validation, controlled SSRF testing, and remediation.
- [Web Exploitation — XXE Local File Read](Practice-Labs/Web/XXE-Local-File-Read.md) — XML parser analysis, external entity resolution, controlled local-file disclosure testing, and remediation.
- [Cryptography — Repeating-Key XOR Analysis](Practice-Labs/Cryptography/Repeating-Key-XOR-Analysis.md) — XOR properties, known-plaintext reasoning, repeating-key recovery, decryption, and validation.
- [Forensics — PNG LSB Steganography](Practice-Labs/Forensics/PNG-LSB-Steganography.md) — PNG metadata triage, RGB least-significant-bit extraction, byte reconstruction, hidden-data validation, and evidence hashing.
- [Linux Privilege Escalation — SUID Misconfiguration](RingZero/Practice-Labs/Linux-Privilege-Escalation/SUID-Misconfiguration.md) — SUID enumeration, binary analysis, unsafe command lookup, PATH manipulation, exploitation flow, and remediation.
- [Reverse Engineering — Static License Check](Practice-Labs/Reverse-Engineering/Static-License-Check.md) — Static binary analysis, input-to-comparison data flow, transformation recovery, GDB reconnaissance, and secure validation design.
- [Network Security — DNS Exfiltration PCAP](Practice-Labs/Network-Security/DNS-Exfiltration-PCAP.md) — DNS traffic analysis, encoded-query detection, fragment reconstruction, PCAP investigation, and defensive monitoring.
- [Forensics — PDF Embedded Object Analysis](Practice-Labs/Forensics/PDF-Embedded-Object-Analysis.md) — PDF object enumeration, embedded-file identification, safe stream extraction, file-type validation, hashing, and forensic handling.

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
