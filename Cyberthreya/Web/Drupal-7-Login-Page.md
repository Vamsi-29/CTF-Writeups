# CyberThreya — Drupal 7 Login Page

## Category

Web Exploitation

## Target

- Challenge host: `192.168.1.13`
- Application: Drupal-based web application
- Environment: CyberThreya CTF

> The target was a CTF machine in an authorized challenge environment.

## Objective

Enumerate the web application, identify the underlying technology and version, find a suitable known vulnerability, gain access to the target, and locate the challenge flag.

## Reconnaissance

I started with Nmap to identify exposed services and gather version information. I also enumerated the web application and manually inspected the discovered paths.

The initial observations included:

- A web application with a login form.
- PHP-based application behavior.
- Multiple accessible paths that provided additional application information.
- A deeper review of the Nmap results revealed that the application was running **Drupal 7**.

The version information was the key pivot: instead of continuing with generic login attacks, I moved toward vulnerability research for the identified CMS/version.

## Vulnerability Research

I searched Metasploit for known Drupal 7 modules and filtered the available results for an exploit applicable to the observed web application and its login-oriented functionality.

The important lesson here was that the CMS/version fingerprint obtained during reconnaissance provided a much narrower attack surface than the initial generic SQL-injection hypothesis.

## Exploitation

1. Identify the web service with Nmap.
2. Enumerate the discovered web paths manually.
3. Confirm the application was PHP-based.
4. Re-examine the Nmap output and identify **Drupal 7**.
5. Search Metasploit for Drupal 7 vulnerabilities/modules.
6. Filter the available modules for one matching the target's observed application characteristics.
7. Test the suitable module against the CTF target.
8. Successful exploitation provided access to the machine.
9. Enumerate the obtained access and locate `flag.txt`.

The original notes confirm that the challenge was completed by obtaining access to the machine and finding `flag.txt`; the flag value itself was not recorded in the source notes, so it is intentionally not fabricated here.

## Key Takeaways

- **Version fingerprinting matters.** Identifying Drupal 7 transformed a broad web-enumeration problem into a targeted vulnerability-research problem.
- **Don't overcommit to the first hypothesis.** A casual SQL-injection attempt against the login form did not work, so reconnaissance continued instead of forcing the same technique.
- **Review scan output carefully.** The useful Drupal version information was noticed during a second examination of the Nmap results.
- **Match exploits to observed conditions.** Searching for vulnerabilities is more effective when the target's technology, version, and application behavior are known.

## Skills Demonstrated

- Nmap reconnaissance
- Web application enumeration
- Technology/version fingerprinting
- CMS vulnerability research
- Metasploit module selection
- Exploitation in an authorized CTF environment
- Post-exploitation enumeration

## Source Notes

This writeup was reconstructed from the author's original CyberThreya notes in this repository. No flag, CVE, exploit module name, or result was added unless it was explicitly present in those notes.
