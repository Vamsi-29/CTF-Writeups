# CyberThreya — Kira Privilege Escalation

## Challenge / Context

Target: `192.168.1.42`

The objective was to obtain access to the target, identify the path from initial access to higher privileges, and retrieve the protected flag.

## 1. Reconnaissance

Started with a basic Nmap scan and identified SSH and HTTP services exposed by the target.

The HTTP service was inspected manually in a browser. `robots.txt` was accessible and revealed an `uploads` directory.

## 2. Information Disclosure

The exposed uploads directory contained multiple files. Two text files disclosed usernames and password-related information.

This provided useful credentials for the SSH service discovered during reconnaissance.

## 3. Initial Access

The SSH service was tested using the credentials discovered during enumeration. The recovered username/password combination allowed successful SSH authentication and access to the target as a normal user.

## 4. Local Enumeration

After gaining access, the filesystem was enumerated and `user.txt` was located. The file contained an encoded value and a hint that became relevant to the privilege-escalation path.

Further directory enumeration also located `flag.txt`, but the file was restricted to privileged users.

## 5. Privilege Escalation Path

The current account could not directly become root, so the local user list was reviewed. The enumeration revealed a user named `kira`. Earlier web content had indicated that Kira was associated with the root account.

Attempting to switch to `kira` required a password. The encoded value found in `user.txt` was therefore revisited.

The value was Base64-encoded. Decoding it produced the password required for the `kira` account.

## 6. Root Access and Flag

After authenticating as `kira`, the account could be used to obtain root privileges. With root access established, the previously restricted `flag.txt` became readable.

## Attack Chain

```text
Nmap
  ↓
HTTP enumeration
  ↓
robots.txt
  ↓
Exposed uploads directory
  ↓
Credential disclosure
  ↓
SSH access
  ↓
Local enumeration
  ↓
Encoded credential in user.txt
  ↓
Base64 decoding
  ↓
Kira account
  ↓
Root privileges
  ↓
flag.txt
```

## Key Techniques

- Network service enumeration with Nmap
- Web content and `robots.txt` enumeration
- Information disclosure through exposed files
- Credential reuse for SSH access
- Linux post-exploitation enumeration
- Base64 decoding
- Local privilege escalation through an alternate privileged account

## Lessons Learned

1. `robots.txt` should not be treated as a security control; it can reveal useful paths during authorized assessments.
2. Exposed upload directories can disclose credentials or other sensitive information.
3. Credentials discovered during web enumeration should be assessed against other exposed services such as SSH in a controlled CTF environment.
4. Encoded data in challenge artifacts should be inspected when it may contain authentication material or other clues.
5. Privilege escalation often depends on correlating information collected during multiple stages of enumeration rather than relying on a single exploit.

> This writeup documents an authorized CyberThreya CTF environment based on the user's original solution notes. The original notes confirm the successful SSH access, Base64 decoding, Kira account access, root access, and flag retrieval.