# Practice Lab — TCP Service Enumeration and Banner Analysis

> **Self-created CTF-style practice scenario.** This is a lab exercise, not an official CTF challenge or a record of a real-world compromise.

## Category

Network Security

## Objective

Enumerate a controlled host, identify exposed TCP services, analyze service banners, and determine which service provides the intended lab clue.

## Lab Context

The practice target represents a small internal server with several deliberately exposed services. The objective is to approach the host methodically rather than immediately running exploitation tools.

No real credentials or production systems are involved.

## Reconnaissance

Start with a TCP port scan against the authorized lab address:

```bash
nmap -sV -Pn <LAB_IP>
```

The `-sV` option requests service/version detection. The important first step is to record which ports are open and what application appears to be listening.

For a focused follow-up, scan only the discovered ports:

```bash
nmap -sV -p 22,80,8080 <LAB_IP>
```

> The port numbers above are illustrative lab ports. Do not assume that every target exposes the same services.

## Analysis

Suppose the scan shows three TCP services:

```text
22/tcp    open  ssh
80/tcp    open  http
8080/tcp  open  http-proxy
```

The next step is to validate the HTTP services instead of treating the version string as proof of the exact application:

```bash
curl -i http://<LAB_IP>/
curl -i http://<LAB_IP>:8080/
```

Check response headers, status codes, page titles, redirects, and any application-specific information.

For the SSH service, banner collection can be performed with Nmap or Netcat:

```bash
nmap -sV -p 22 <LAB_IP>
nc -nv <LAB_IP> 22
```

The purpose is identification, not credential guessing.

## Vulnerability / Technique

The technique in this practice scenario is **service enumeration and banner analysis**.

A common assessment mistake is to treat an open port as an exploitable vulnerability. An open service is only an observation. The analyst still needs to determine:

1. What protocol is actually running.
2. Which application implements the protocol.
3. Whether the exposed functionality is expected.
4. Whether the detected version is reliable.
5. Whether additional enumeration is justified.

## Solution Steps

### 1. Discover the attack surface

```bash
nmap -Pn <LAB_IP>
```

Record all discovered TCP ports.

### 2. Identify services

```bash
nmap -sV -p- <LAB_IP>
```

Compare the service names and versions with the behavior observed during manual testing.

### 3. Inspect HTTP responses

```bash
curl -i http://<LAB_IP>/
curl -i http://<LAB_IP>:8080/
```

Look for redirects, unusual headers, application names, and endpoints exposed by the lab.

### 4. Validate the interesting service

If a service exposes a text banner or diagnostic endpoint in the controlled lab, inspect it directly rather than assuming the Nmap fingerprint is complete:

```bash
printf 'GET / HTTP/1.1\r\nHost: <LAB_IP>\r\nConnection: close\r\n\r\n' | nc <LAB_IP> 8080
```

### 5. Document the finding

A useful assessment note should contain the port, protocol, observed service, evidence, and next investigation step.

Example:

```text
Port: 8080/tcp
Protocol: HTTP
Evidence: HTTP response confirmed with curl
Next step: enumerate application paths and review exposed functionality
```

## Result

The intended outcome of this practice lab is a **validated service inventory and investigation plan**. No real flag, credential, vulnerability, or production compromise is claimed.

## Lessons Learned

- Start with enumeration before exploitation.
- `-sV` provides useful fingerprints but should be validated against actual service behavior.
- An open port is not automatically a vulnerability.
- HTTP headers and response behavior can reveal more than a simple port scan.
- Keep reconnaissance findings separate from confirmed vulnerabilities.
- Document evidence so another analyst can reproduce the observation.

## Defensive Perspective

Organizations should minimize unnecessary exposed services, restrict administrative interfaces with network controls, remove verbose banners where practical, and continuously inventory externally reachable services.

For security assessments, service enumeration should feed into risk-based validation rather than automatically triggering exploitation.

## Practice-Only Notice

This document is intentionally **self-created practice content**. It does not represent an official CTF challenge, real flag, CVE, ranking, penetration-test result, or personal achievement.
