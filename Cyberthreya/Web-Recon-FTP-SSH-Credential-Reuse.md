# CyberThreya — Web Recon, FTP Access, and SSH Credential Reuse

**Category:** Web / Network Security  
**Technique:** Information disclosure, credential brute-force, credential reuse  
**Platform:** CyberThreya

## Challenge / Context

This challenge involved a target at `192.168.1.39` exposing a web application along with FTP and SSH services. The objective was to enumerate the target, identify useful information exposed by the web application, obtain valid credentials, and retrieve the challenge flag.

> The target was a CTF machine in an authorized challenge environment.

## Reconnaissance / Analysis

The original solve notes record the following reconnaissance path:

1. A basic Nmap scan identified multiple open ports.
2. The target was running a web application, which was manually inspected for additional information.
3. The page source contained a comment revealing the username `steve`.
4. FTP and SSH were both exposed.
5. The page source also contained an FTP-related reference, making the FTP service a useful next target.

The important observation was that the web application did not directly provide the flag, but it disclosed information that could be chained with the exposed network services.

## Vulnerability / Technique

The challenge demonstrates a combination of weaknesses rather than a single application vulnerability:

- **Information disclosure:** a username was exposed in HTML source comments.
- **Weak credential security:** the discovered username was successfully tested against `rockyou.txt` for FTP authentication.
- **Credential reuse:** the same credentials worked for SSH after FTP access was obtained.
- **Exposed services:** both FTP and SSH expanded the attack surface.

The original notes explicitly record that Hydra was used with the `steve` username and `rockyou.txt`, and that the password discovered was `Andrew`.

## Exploitation / Solution

### 1. Enumerate the target

A basic Nmap scan was used to identify exposed services. The important findings were the web application, FTP, and SSH.

### 2. Inspect the web application

The web page source was reviewed manually. A comment in the source disclosed the username:

```text
steve
```

The source also referenced FTP, so the FTP service became the next point of investigation.

### 3. Test FTP credentials

The original solve used Hydra with `rockyou.txt` against the FTP service and the discovered username.

The successful credential was recorded as:

```text
Username: steve
Password: Andrew
```

### 4. Access FTP and locate the flag

Using the recovered credentials, FTP access was obtained. The directory contained:

```text
flag.txt
```

However, the notes record that the file could not be downloaded because the FTP account did not have sufficient permission.

This was an important pivot point: instead of treating the FTP restriction as the end of the challenge, the other exposed service was revisited.

### 5. Reuse the credentials against SSH

The same username/password pair was tested against SSH. The original notes confirm that the credentials were reused successfully:

```text
Username: steve
Password: Andrew
```

SSH access provided a shell on the target system, where `flag.txt` could be accessed successfully.

## Result

The challenge was successfully completed and the flag was retrieved through the SSH session.

The original source notes do **not** record the flag value, so the flag is intentionally omitted rather than fabricated.

## Lessons Learned

- Review HTML source, not just the rendered page; comments can disclose useful reconnaissance information.
- Enumerate all exposed services before committing to a single attack path.
- A credential that works on one service should be tested against other exposed services when credential reuse is plausible.
- An access-control restriction on one service does not necessarily prevent the same underlying account from being useful elsewhere.
- The strongest CTF workflow was the chain: **web information disclosure → credential discovery → FTP access → credential reuse → SSH access → flag retrieval**.

## Skills Demonstrated

- Nmap-based service enumeration
- Web source-code reconnaissance
- Information disclosure analysis
- Hydra-based credential testing in a CTF environment
- FTP enumeration
- SSH authentication
- Credential-reuse analysis
- Basic Linux post-authentication enumeration

## Source Notes

This writeup was reconstructed from the author's original CyberThreya notes stored in the repository. The username, password, service findings, FTP permission issue, SSH reuse, and successful flag retrieval are taken directly from those notes. The flag value is intentionally not supplied because it was not present in the source material.
