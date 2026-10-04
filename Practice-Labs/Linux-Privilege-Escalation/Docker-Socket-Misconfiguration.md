# CTF-Style Practice Lab — Docker Socket Misconfiguration

> **Practice content:** This is a self-created lab scenario for learning Linux privilege-escalation methodology. It is not an official CTF challenge and does not represent a real compromise, flag, ranking, CVE, or personal achievement.

## Challenge / Context

A low-privileged user on a Linux lab host can interact with the local Docker daemon. The task is to determine whether that access crosses a privilege boundary and, if so, demonstrate the impact safely inside the authorized lab.

The lab is intentionally designed around a common administrative mistake: exposing the Docker Unix socket to a user who should not have Docker-admin privileges.

No real credentials, secrets, production hosts, or external targets are involved.

## Reconnaissance

Start with basic identity and group enumeration.

```bash
id
groups
uname -a
```

Look for Docker-related group membership:

```bash
getent group docker
```

Then check whether the Docker client can communicate with the daemon:

```bash
docker version
docker info
```

A successful daemon query is important evidence. Merely having the `docker` client installed does not imply administrative access.

## Vulnerability / Technique

The security boundary is the Docker daemon.

A user who can control a privileged Docker daemon can generally request operations that affect the host, including starting containers with powerful filesystem or namespace settings. Therefore, membership in a local `docker` group should be treated as **high privilege**, not as an ordinary application permission.

The lab focuses on recognizing this trust relationship rather than attacking a remote Docker service.

## Solution Steps

### 1. Confirm the effective permissions

Record the current user and groups:

```bash
id
groups
```

Expected lab observation:

```text
uid=1001(labuser) gid=1001(labuser) groups=1001(labuser),999(docker)
```

The exact UID/GID values are lab-specific and are not credentials.

### 2. Confirm Docker daemon access

```bash
docker info
```

If this succeeds as the low-privileged account, the user can issue requests to the daemon.

### 3. Enumerate available images

```bash
docker images
```

For a controlled lab, use only an image explicitly supplied by the lab environment.

### 4. Demonstrate the privilege boundary safely

Instead of accessing arbitrary host files, create a disposable lab marker on the host:

```bash
sudo sh -c 'printf "docker-lab-marker\n" > /root/docker-lab-marker'
```

The command above is performed by the **lab administrator while preparing the scenario**, not by the learner.

The learner can then use a disposable container configured by the lab to demonstrate access to the marker through a controlled bind mount:

```bash
docker run --rm -v /root:/host-root LAB_IMAGE cat /host-root/docker-lab-marker
```

If the marker is readable from the container, the exercise demonstrates that Docker-daemon control can cross the host filesystem boundary.

> **Important:** In a real assessment, do not read unrelated host files or modify the host. Use a dedicated marker or other agreed test artifact.

### 5. Record the evidence

Useful evidence includes:

```bash
id
getent group docker
docker info
docker images
docker ps -a
```

Capture the commands and output in the lab notes so the finding is reproducible.

## Why This Works

The Docker daemon normally runs with root-level host privileges. A user who can issue unrestricted Docker API requests can ask the daemon to perform operations on that user's behalf.

The important chain is:

```text
Low-privileged user
        |
        v
Membership / access to docker group
        |
        v
Docker daemon control
        |
        v
Privileged container operations
        |
        v
Potential host-level impact
```

The weakness is therefore not a bug in the container image. It is an authorization/design problem caused by granting control of a privileged daemon to an otherwise untrusted user.

## Result

**Practice result:** the lab demonstrates that unrestricted Docker-daemon access is a privilege boundary and can provide host-level impact through controlled container operations.

No real system was compromised, and no real flag or external target is claimed.

## Defensive Recommendations

- Do not add untrusted users to the `docker` group.
- Treat Docker-daemon access as privileged administrative access.
- Prefer rootless container runtimes where appropriate.
- Restrict access to Docker's Unix socket.
- Monitor membership changes to privileged container-management groups.
- Use authorization controls when exposing container APIs.
- Separate developer workloads from sensitive host resources.
- Do not expose the Docker daemon over an unauthenticated TCP socket.

## Lessons Learned

- Always enumerate groups during Linux privilege-escalation reconnaissance.
- Installed tooling and actual daemon permissions are different things.
- A Unix socket can represent a major privilege boundary.
- Containerization does not automatically isolate a privileged container runtime from the host.
- Validate impact with a dedicated lab marker instead of accessing unrelated sensitive files.
- Document the authorization path, not just the final effect.

## Practice Classification

This document is intentionally stored under `Practice-Labs/` to distinguish it from verified CTF writeups. It is self-created training content for practicing Linux privilege-escalation analysis in an authorized lab.
