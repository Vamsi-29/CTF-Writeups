# CyberThreya — Web Source Disclosure and FTP/SSH Credential Reuse

## Challenge / Context

Target: `192.168.1.39`

This challenge involved enumerating a host running a web service, extracting information from the page source, obtaining access to FTP, and then using the recovered credentials to access SSH and retrieve the flag.

## Reconnaissance

A basic Nmap scan identified several open ports. The host was running a website, and both FTP and SSH were exposed.

The web page was then inspected, including its HTML source.

## Information Disclosure

A username, `steve`, was present in a comment in the page source. The source also contained a reference to FTP, making the exposed FTP service a useful next investigation point.

This demonstrates why security testing should include both the rendered page and underlying source: comments and developer artifacts can unintentionally disclose operational information.

## FTP Access

The discovered username was tested against the FTP service using the credential-testing approach documented in the original solve notes. Valid credentials were recovered.

Using those credentials, FTP access was obtained and `flag.txt` was found. However, the file could not be downloaded through FTP because the account did not have sufficient permission.

## SSH Credential Reuse

The recovered credentials were then tested against SSH. The same username/password combination worked, allowing an SSH session to the system.

From the SSH session, `flag.txt` was successfully retrieved.

## Attack Chain

```text
Nmap enumeration
      ↓
Web application discovery
      ↓
HTML source inspection
      ↓
Username disclosure
      ↓
FTP access
      ↓
flag.txt discovered but restricted
      ↓
Credential reuse against SSH
      ↓
SSH access
      ↓
Flag retrieval
```

## Security Lessons

1. **Do not expose operational information in page source.** HTML comments can disclose usernames, service references, or development details.
2. **Avoid password reuse across services.** Reusing credentials across FTP and SSH can turn limited service access into broader host access.
3. **Use service-specific credentials and strong authentication.** SSH should preferably use keys or another stronger authentication mechanism.
4. **Review externally accessible services.** Unnecessary FTP/SSH exposure increases attack surface.
5. **Permissions are defense in depth.** The FTP restriction limited direct flag retrieval, but did not prevent the credentials from being reused against SSH.

## Techniques Demonstrated

- Network/service enumeration
- Web source-code inspection
- Information disclosure
- Credential discovery
- FTP authentication testing
- SSH authentication testing
- Credential reuse
- Post-authentication file retrieval

## Source

This writeup is based on the author's existing CyberThreya solve notes in this repository. No challenge, exploit path, or result has been fabricated.
