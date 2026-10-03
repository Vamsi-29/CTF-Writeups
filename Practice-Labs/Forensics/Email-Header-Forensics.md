# CTF-Style Practice Lab — Email Header Forensics

> **Practice content:** This is a self-created lab scenario for learning email-header analysis. It is not an official CTF challenge and does not represent a real incident, flag, ranking, or personal achievement.

## Challenge / Context

A fictional security team receives a suspicious email and needs to determine where it came from and whether the message was likely spoofed.

The objective is to reconstruct the mail path from the headers, identify the relevant authentication results, compare the visible sender with the authenticated sender, and document the evidence.

The scenario uses sanitized lab headers only. No real email addresses, credentials, domains, or personal information are used.

## Reconnaissance / Analysis

The first step is to preserve the complete raw message rather than relying only on the visible From, To, and Subject fields.

Useful evidence includes `Received`, `Return-Path`, `Authentication-Results`, `Received-SPF`, `DKIM-Signature`, and `Message-ID`.

Sanitized example:

```text
From: security-alert@example.test
Return-Path: bounce@example.test
Subject: Account review

Received: from mail.example.test (192.0.2.20)
        by mx.example.test with ESMTPS;
        Sat, 03 Oct 2026 08:15:00 +0000

Authentication-Results: mx.example.test;
        spf=pass smtp.mailfrom=example.test;
        dkim=pass header.d=example.test;
        dmarc=pass header.from=example.test
```

## Technique

Email-header analysis is primarily a **correlation problem**. A single header should not be treated as proof of origin. Correlate the chronological `Received` chain, SMTP envelope sender, SPF, DKIM signing domain, visible From domain, DMARC alignment, Message-ID, and timestamps.

## Solution Steps

### 1. Preserve the raw message

Save the original message as `suspicious.eml` and avoid modifying the headers before analysis.

### 2. Extract headers

```bash
grep -Ei '^(From|To|Subject|Return-Path|Received|Authentication-Results|Received-SPF|DKIM-Signature|Message-ID):' suspicious.eml
```

### 3. Read the Received chain

`Received` headers are normally added by mail infrastructure as a message moves between systems. For timeline reconstruction, start with the lowest relevant Received header and work upward toward the receiving system.

Record the sending host, source IP, receiving host, timestamp, and protocol.

### 4. Check SPF

Look for `spf=pass` or `spf=fail`. SPF evaluates the sending infrastructure for the SMTP envelope domain. It does not, by itself, prove that the visible From address is legitimate.

### 5. Check DKIM

Inspect the signing domain represented by `d=` in the DKIM-Signature header. A valid DKIM signature associates signed content with that signing domain at verification time; it does not automatically prove that the sender is trustworthy.

### 6. Check DMARC alignment

DMARC connects authentication results with the visible From domain. If SPF and DKIM both fail alignment, the message deserves additional scrutiny.

### 7. Correlate the evidence

| Evidence | Finding | Interpretation |
|---|---|---|
| From | `<DOMAIN>` | Visible sender |
| Return-Path | `<DOMAIN>` | Envelope sender |
| SPF | `<PASS/FAIL>` | Sending-host authentication |
| DKIM | `<PASS/FAIL>` | Cryptographic signing result |
| DMARC | `<PASS/FAIL>` | Domain-alignment result |
| Received chain | `<SUMMARY>` | Delivery path |

## Optional Python Helper

For a local lab message, Python's standard library can parse headers without changing the source file:

```python
from email import policy
from email.parser import BytesParser

with open("suspicious.eml", "rb") as f:
    msg = BytesParser(policy=policy.default).parse(f)

for name in (
    "From", "Return-Path", "Subject", "Authentication-Results",
    "Received-SPF", "DKIM-Signature", "Message-ID",
):
    print(f"{name}: {msg.get(name)}")

print("\\nReceived headers:")
for header in msg.get_all("Received", []):
    print(header)
```

This helper is intentionally limited to evidence extraction; it does not attempt to declare an email malicious automatically.

## Result

**Practice result:** the analyst can reconstruct the sanitized delivery path and correlate SPF, DKIM, DMARC, and sender-domain information to produce an evidence-based assessment.

No real incident, malicious sender, real-world attribution, or confirmed compromise is claimed.

## Lessons Learned

- Preserve the original message before analysis.
- Read the complete Received chain instead of trusting the visible From field.
- SPF, DKIM, and DMARC answer different questions and should be correlated.
- Authentication success does not automatically mean an email is safe.
- Header analysis is strongest when every conclusion is tied to observable evidence.
- UTC timestamps make cross-system timeline reconstruction easier.

## Defensive Recommendations

- Configure SPF, DKIM, and DMARC for organizational domains.
- Monitor DMARC reports for unexpected sending infrastructure.
- Preserve mail headers during phishing investigations.
- Train analysts to distinguish envelope sender, signing domain, and visible From domain.
- Combine header evidence with URL, attachment, endpoint, and identity telemetry before declaring an incident.

## Practice Classification

This document is intentionally stored under `Practice-Labs/` to distinguish it from verified CTF writeups. It is a self-created training scenario for practicing practical email-forensics methodology.
