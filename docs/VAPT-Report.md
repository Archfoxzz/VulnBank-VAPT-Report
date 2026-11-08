# VulnBank – Web Application VAPT Report

**Report status:** Evidence-based draft  
**Assessment evidence dates:** 27 October 2026 and 8 November 2026  
**Target shown in supplied evidence:** `https://vulnbank.org`  
**Classification:** Portfolio report – review authorization and disclosure restrictions before publication

> **Scope caveat:** This report is based on screenshots supplied by the tester. It does not claim independent reproduction, complete coverage, or confirmed authorization. Confirm written permission, in-scope assets, test window, and rules of engagement before external distribution.

## 1. Executive summary

The strongest evidence is a directory-enumeration result showing `/.env` returning HTTP 200 with an approximately 571-byte response, together with a screenshot displaying application configuration, database and Redis connection settings, an API key, and SMTP configuration. If this file was retrievable without authentication from the assessed target, this is a **Critical sensitive-information disclosure**.

Other screenshots show responding application routes, a successful login exchange, a decoded JWT containing an administrative claim, an administrative interface, and an unusually large displayed balance. These observations do **not** independently prove unauthorized administrative access, JWT forgery, IDOR, or server-side financial manipulation. They require controlled validation.

## 2. Scope and methodology represented

### Activities represented by the evidence

- Nmap service discovery.
- Web path discovery / directory enumeration.
- Browser-based application inspection.
- Burp Suite HTTP request and response review.
- JWT structure decoding.

### Evidence limitations

No complete test log, full raw HTTP export, authenticated role matrix, server logs, source review, or retest evidence was supplied. The report therefore separates findings from observations requiring further validation.

## 3. Findings

### VULN-01 – Public exposure of application environment configuration

| Attribute | Assessment |
|---|---|
| Severity | **Critical – provisional** |
| Category | Sensitive information disclosure / security misconfiguration |
| Affected path | `https://vulnbank.org/.env` |
| Evidence | Enumeration output shows HTTP 200 and approximately 571 bytes; a screenshot displays sensitive configuration variable names and values. |
| Confidence | High that the screenshot depicts sensitive configuration content; unauthenticated reachability and credential validity should be independently confirmed. |

**Description:** The enumeration output reports a successful response for `/.env`. The associated screenshot displays application secret variables, database connection details, Redis settings, a third-party API key, and SMTP configuration. Secret values are intentionally excluded from this repository.

**Potential impact:**

- Exposed signing keys may allow token forgery depending on application implementation.
- Database or Redis credentials may permit unauthorized access if the services are reachable and the credentials have sufficient privileges.
- SMTP credentials may enable abuse of the mail service.
- Third-party API keys may permit unauthorized use of connected services.
- Production debug mode increases information-disclosure risk.

**Recommendations:**

1. Remove the file from the web document root and deny access to dotfiles at the web server/CDN layer.
2. Rotate all potentially exposed credentials and keys; invalidate sessions and tokens signed with exposed keys.
3. Store secrets in a managed secret store or protected deployment environment.
4. Disable debug mode in production and add deployment checks.
5. Review logs for successful requests to `/.env` and related variants.

**Evidence:** [Directory enumeration](../evidence/01-directory-enumeration.png) · [Redacted environment screenshot](../evidence/02-exposed-env-redacted.png)

### VULN-02 – Application routes and diagnostic interfaces exposed for review

| Attribute | Assessment |
|---|---|
| Severity | Informational pending validation |
| Paths observed | `/api/docs`, `/console`, `/healthz`, `/graphql` |
| Evidence | Directory enumeration reports responding routes. |
| Confidence | High for the response observations; security impact is not established. |

A successful HTTP response alone does not establish a vulnerability. Review whether these routes disclose internal schemas, operational details, diagnostic information, or administrative functionality.

**Recommendations:**

- Require authentication and role-based authorization for non-public documentation and console interfaces.
- Limit health endpoints to minimal liveness/readiness information.
- Restrict GraphQL introspection in production where unnecessary; enforce object-level authorization and query limits.
- Confirm whether `/console` is intended to be publicly accessible.

**Evidence:** [Directory enumeration](../evidence/01-directory-enumeration.png)

### VULN-03 – Session cookie attributes require verification

| Attribute | Assessment |
|---|---|
| Severity | Low / hardening observation |
| Evidence | Captured `Set-Cookie` fragment shows `HttpOnly` and `Path=/`; `Secure` and `SameSite` are not visible in the screenshot. |
| Confidence | Medium; inspect the complete raw response header before confirming missing attributes. |

**Recommendations:** Set `Secure`, retain `HttpOnly`, and configure an appropriate `SameSite` value. Use suitable session expiry and revocation. The screenshot may omit header content, so this is not a confirmed missing-attribute finding.

**Evidence:** [Redacted login exchange](../evidence/03-login-exchange-redacted.png)

## 4. Observations requiring validation

### Administrative interface and JWT role claim

The screenshots show an administrative user-management interface and, separately, a decoded JWT payload containing an administrative claim. Decoding a JWT does not verify its signature. These screenshots do not prove that a standard user accessed administrator functionality.

**Validation approach:** With explicitly authorized test accounts, verify server-side role enforcement, signature validation, algorithm allowlisting, issuer/audience checks, expiry, and trusted role assignment. Begin with read-only access-control checks.

**Evidence:** [Redacted admin interface](../evidence/05-admin-panel-redacted.png) · [Redacted JWT debugger](../evidence/06-jwt-debugger-redacted.png)

### Unusually large displayed account balance

A dashboard screenshot displays an unusually large balance. A UI value alone is not proof of backend manipulation. Confirm the server-returned value and persistence after re-login, and verify that transaction limits and available-funds checks are enforced server-side. Use test accounts only; do not initiate real transfers.

**Evidence:** [Redacted dashboard](../evidence/07-dashboard-redacted.png)

### Login API response

The captured `POST /login` exchange returns HTTP 200 with a success status, token, and account attributes. This is consistent with successful authentication and does not establish authentication bypass. Review whether unnecessary sensitive fields are returned and whether failed logins are handled consistently.

**Evidence:** [Redacted login exchange](../evidence/03-login-exchange-redacted.png) · [Redacted login request](../evidence/04-login-request-redacted.png)

### Network/service discovery

The Nmap screenshot reports HTTP-related services on ports 80, 443, and 8080, with Cloudflare identified as the HTTP proxy. This should not be treated as a definitive inventory of origin-server services.

**Evidence:** [Nmap results](../evidence/08-nmap-results.png)

## 5. Remediation priority

| Priority | Action | Suggested timeframe |
|---|---|---|
| P0 | Remove `.env` exposure; rotate potentially exposed secrets; invalidate affected sessions/tokens. | Immediately |
| P1 | Review logs for access and credential misuse; assess dependent-service exposure. | 24–48 hours |
| P1 | Disable production debug mode and improve secret storage. | Within 3 days |
| P2 | Review API docs, console, GraphQL, and health endpoint access controls. | Within 7 days |
| P2 | Verify cookie attributes and session lifecycle. | Within 7 days |
| P3 | Validate role authorization and server-side balance integrity using test accounts. | Authorized retest |

## 6. Retest checklist

- [ ] Unauthenticated `/.env` requests return 403/404 and no cached copies remain.
- [ ] All exposed secrets are rotated; old sessions and tokens are invalidated.
- [ ] Production debug mode is disabled.
- [ ] Documentation, console, GraphQL, and health endpoints expose only intended information.
- [ ] Standard users cannot invoke administrative functions.
- [ ] Invalid or tampered JWTs are rejected server-side.
- [ ] Session cookies use `Secure`, `HttpOnly`, and an appropriate `SameSite` value.
- [ ] Balances and transaction limits are enforced server-side.
- [ ] Logs show no unexplained access to configuration files or credential misuse.

## 7. Limitations and responsible disclosure

- This is a screenshot-based portfolio write-up, not a complete independent audit.
- No origin-server inventory is established by the Cloudflare-facing Nmap evidence.
- No successful JWT forgery, privilege escalation, IDOR, SQL injection, RCE, or financial transaction manipulation is claimed.
- Confirm authorization and the target owner's disclosure policy before publishing identifying details.
- Keep original unredacted evidence in a private, access-controlled location. Never commit raw tokens, passwords, API keys, cookies, database connection strings, or personal account data.

## 8. Disclaimer

This material is for educational and portfolio purposes. It is not authorization to test the target. Only assess systems you own or have explicit written permission to test.
