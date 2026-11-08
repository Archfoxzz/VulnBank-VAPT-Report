# VulnBank – Web Application VAPT

> **Portfolio / educational security assessment.** This repository documents an evidence-based assessment of the VulnBank web application using the screenshots collected during testing.

## Project overview

This project demonstrates a structured web application Vulnerability Assessment and Penetration Testing (VAPT) workflow: reconnaissance, web content discovery, HTTP traffic inspection, evidence handling, risk classification, and remediation reporting.

The report deliberately distinguishes **observed evidence** from **issues that still require validation**. A decoded JWT, a visible admin page, or an unusual balance display is not by itself proof of privilege escalation, token forgery, or financial logic manipulation.

## Scope represented in the evidence

- Target shown in screenshots: `https://vulnbank.org`
- Activities represented: Nmap service discovery, directory/content enumeration, browser inspection, Burp Suite request/response review, and JWT structure inspection.
- Evidence dates visible in screenshots: 27 October 2026 and 8 November 2026.
- This repository is a retrospective portfolio write-up based on supplied screenshots, not a claim of a complete independent audit.

**Authorization note:** Only assess systems you own or have explicit written permission to test. Before publishing, confirm that the target owner permits public disclosure of the hostname, findings, and screenshots. If not, replace the target with a fictionalized name and remove identifying details.

## Key observations

| ID | Observation | Status / severity |
|---|---|---|
| VULN-01 | Publicly retrievable environment configuration (`/.env`) containing sensitive configuration values | **Critical – provisional**, subject to confirming unauthenticated reachability and secret validity |
| VULN-02 | Application routes such as API documentation, console, health, and GraphQL endpoints respond | Informational pending access-control and content review |
| VULN-03 | Session cookie attributes need full-header verification | Low / hardening observation; not confirmed |
| OBS-01 | Admin interface and JWT with an administrative claim are visible in separate screenshots | Requires authorization and token-validation testing |
| OBS-02 | Dashboard displays an unusually large balance | Requires server-side business-logic validation |
| OBS-03 | HTTP-related services are observed behind Cloudflare | Reconnaissance observation, not an origin-server inventory |

## Repository structure

```text
VulnBank-VAPT/
├── README.md
├── .gitignore
├── report/
│   └── VulnBank-VAPT-Report.pdf
├── docs/
│   └── VAPT-Report.md
└── evidence/
    ├── 01-directory-enumeration.png
    ├── 02-exposed-env-redacted.png
    ├── 03-login-exchange-redacted.png
    ├── 04-login-request-redacted.png
    ├── 05-admin-panel-redacted.png
    ├── 06-jwt-debugger-redacted.png
    ├── 07-dashboard-redacted.png
    └── 08-nmap-results.png
```

## Tools represented

- **Nmap** – service discovery and basic service identification.
- **Directory enumeration tool** – discovery of web paths and response codes.
- **Burp Suite Community Edition** – HTTP request/response inspection.
- **Browser Developer Tools** – application storage and network inspection.
- **JWT.io debugger** – decoding JWT structure for inspection. Decoding does not verify a token's signature.

## Important security and publication notes

- Evidence images in this repository are redacted. Do not add original screenshots, raw HTTP exports, cookies, JWTs, passwords, API keys, database connection strings, or real account data.
- The PDF report is a draft and uses cautious language where the screenshots do not prove exploitability.
- Treat any credentials that appeared in the original environment-file screenshot as compromised if they are real. Rotate them and invalidate affected sessions.
- Do not publish proof-of-concept exploit steps or sensitive details unless disclosure is authorized.
- No successful JWT forgery, privilege escalation, IDOR, SQL injection, remote code execution, or financial transaction manipulation is claimed.

## How to use this repository

1. Read the [VAPT report](docs/VAPT-Report.md) for the detailed findings and remediation recommendations.
2. Review the redacted evidence in `evidence/`.
3. Use the PDF version in `report/` for a formatted report.
4. Refer to the limitations and retest checklist before treating any item as a confirmed vulnerability.

## Disclaimer

This material is provided for educational and portfolio purposes. It is not a guarantee that the target is vulnerable, and it is not authorization to test the target. The author is not responsible for unauthorized use of this material.
