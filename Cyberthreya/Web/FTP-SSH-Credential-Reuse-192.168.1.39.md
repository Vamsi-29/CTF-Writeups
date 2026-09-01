# CyberThreya — Web Recon, FTP Access, and SSH Credential Reuse

**Category:** Web / Network Security  
**Techniques:** Web source inspection, FTP enumeration, credential reuse, SSH access

## Challenge / Context

The target was a CyberThreya lab machine at `192.168.1.39` exposing a web service along with FTP and SSH.

The objective was to enumerate the exposed services, identify useful information from the website, gain access to the available services, and retrieve the flag.

## Reconnaissance / Analysis

I started with a basic Nmap scan to identify the exposed services.

The scan showed that the target was running a website and had both **FTP** and **SSH** available.

I then opened the website and inspected the page source.

A comment in the source code exposed a username:

```text
steve
```

The source also contained a reference to FTP, which made the FTP service a useful next target.

## Vulnerability / Technique

The main issues identified during enumeration were:

- Sensitive information exposed in HTML source comments.
- FTP exposed as an accessible service.
- A valid username disclosed by the web application.
- Password-based authentication susceptible to credential guessing.
- Reuse of the same credentials between FTP and SSH.

The combination of information disclosure and credential reuse provided a path from the public web service to authenticated system access.

## Exploitation / Solution

### 1. Enumerate the target

A basic Nmap scan was used to identify open services.

```bash
nmap -sV 192.168.1.39
```

The scan identified the web service, FTP, and SSH.

### 2. Inspect the website source

The website source was reviewed manually.

A comment contained the username:

```text
steve
```

The source also referenced FTP, so I focused on the FTP service next.

### 3. Test FTP authentication

The discovered username was tested against the FTP service using a password wordlist in the authorized lab environment.

```bash
hydra -l steve -P /path/to/rockyou.txt ftp://192.168.1.39
```

The original solve notes confirm that valid FTP credentials were recovered.

### 4. Access FTP

The recovered credentials were used to authenticate to FTP.

```bash
ftp 192.168.1.39
```

A `flag.txt` file was visible on the FTP server, but the file could not be downloaded because the FTP account did not have sufficient permission.

This indicated that FTP access alone was not enough to retrieve the flag.

### 5. Test credential reuse on SSH

Because SSH was also exposed, the recovered credentials were tested against SSH in the authorized lab environment.

```bash
hydra -l steve -P /path/to/rockyou.txt ssh://192.168.1.39
```

The original solve notes confirm that the same username/password combination worked for SSH.

### 6. Retrieve the flag through SSH

I connected to the target using SSH:

```bash
ssh steve@192.168.1.39
```

After logging in, the previously inaccessible file could be read from the system:

```bash
cat flag.txt
```

The original solve notes confirm successful flag retrieval. The flag value is intentionally not repeated here.

## Result

The complete attack path was:

```text
Web Enumeration
      ↓
Source Code Inspection
      ↓
Username Disclosure
      ↓
FTP Enumeration
      ↓
Credential Discovery
      ↓
FTP Access
      ↓
Permission Restriction
      ↓
Credential Reuse on SSH
      ↓
SSH Access
      ↓
Flag Retrieval
```

## Lessons Learned

- Always inspect HTML source code during web reconnaissance.
- Comments can unintentionally disclose usernames and other useful information.
- Enumerate all exposed services instead of focusing only on the web application.
- Credentials discovered for one service should be considered compromised and should not be reused elsewhere.
- Different services can enforce different permissions for the same account.
- FTP access does not necessarily provide the same level of access as an interactive SSH session.
- Service enumeration and correlating information across services can reveal the complete attack path.

## Security Recommendations

- Remove usernames and operational information from public HTML comments.
- Disable unnecessary FTP services and prefer secure alternatives such as SFTP.
- Do not reuse passwords between services.
- Use strong, unique credentials and appropriate authentication controls.
- Apply least-privilege permissions to service accounts.
- Monitor authentication failures and suspicious credential-guessing activity.

## Source Notes

This writeup is based on the existing `target -- 192.168.1.39 (webpage with username).txt` solve notes already stored in this repository. The target, username disclosure, exposed services, FTP/SSH workflow, permission issue, credential reuse, and successful flag retrieval are taken from those notes. Credential values and the flag are intentionally not repeated in this polished writeup.
