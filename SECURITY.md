# Security Policy

SelfHostHub runs software on people's home laptops and holds their personal files, budgets and
media. We take security seriously and appreciate responsible disclosure.

## Supported Versions

SelfHostHub runs entirely on the user's own laptop. We operate no servers. Only the latest
release and the current `main` branch receive security fixes. The app updates itself.

| Version | Supported |
|---------|-----------|
| Latest release / `main` | ✅ |
| Older releases | ❌ |

## Reporting a Vulnerability

**Do not open a public GitHub issue for security vulnerabilities.**

Report privately using **GitHub's private vulnerability reporting** — the
[**Security → Report a vulnerability**](https://github.com/ginutgeorge-Aus/selfhosthub/security/advisories/new) tab on this repository.

Include:
- A description of the vulnerability
- Steps to reproduce
- Potential impact
- Any suggested fix (optional)

## Response Timeline

| Stage | Target |
|-------|--------|
| Acknowledgement | 3 business days |
| Initial assessment | 7 business days |
| Fix or mitigation | 30 days (critical: 7 days) |

We will coordinate a disclosure timeline with you and credit reporters who wish to be named.

## Scope

In scope:
- The Windows app, its installer and its updater (code execution, privilege escalation, update tampering)
- `hub-agent` and its local API (unauthenticated access, command injection)
- Exposure of apps or the dashboard beyond the home network
- Leaking or mishandling TLS private keys or DuckDNS/deSEC tokens
- App catalog tampering (manifest or image substitution)

Out of scope:
- Theoretical vulnerabilities with no practical exploit path
- Issues requiring physical access to the server
- Social engineering attacks
- Vulnerabilities inside a hosted third-party app (Jellyfin, Actual, …). Report those upstream.
- Issues in DuckDNS, deSEC or Let's Encrypt themselves

## For users

- Keep the laptop's Windows updates on, and let SelfHostHub update itself.
- Don't forward router ports to the laptop. SelfHostHub is designed for the home network only.
