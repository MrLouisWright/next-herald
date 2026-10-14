# Security policy

## Supported versions
next-herald is pre-alpha and **no version has been published**. Once releases exist, the latest minor of the latest major receives security fixes.

| Version | Supported |
|---|---|
| unreleased (`main`) | ✅ best effort |

## Reporting a vulnerability
Please **do not open a public issue.** Use GitHub's [private vulnerability reporting](https://github.com/MrLouisWright/next-herald/security/advisories/new). You'll get an acknowledgement within 7 days and a fix or mitigation plan within 30 days for confirmed issues. Credit is given in the advisory unless you prefer otherwise.

## Threat model
See [docs/security.md](docs/security.md). Highest-risk areas: JSON-LD serialization (XSS), XML writing, canonical URL generation (host-header poisoning) and `herald check` network access.
