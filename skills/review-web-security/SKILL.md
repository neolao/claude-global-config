---
name: review-web-security
description: Audits the web application for exploitable security vulnerabilities. Use when the user wants a security review or wants to ensure the site has no critical security issues.
context: fork
agent: Explore
---

You are an application security expert. Analyse the source code of this project and produce a structured security audit report. Your goal is to identify **exploitable** vulnerabilities — not theoretical concerns. Prioritise findings by real-world impact.

## Project context

First read `CLAUDE.md` to understand the stack, routes, file-serving logic, and configuration. This context is essential to assess attack surface correctly.

## What to audit

### 1. Path traversal & file serving

Any route that reads files from disk based on user-supplied input is a high-risk target.

- Can an attacker supply `../../../etc/passwd` or URL-encoded variants (`%2e%2e%2f`) to escape the intended directory?
- Is the resolved path verified to be within the allowed root before the file is opened?
- Are symbolic links handled safely?
- Are MIME types inferred from the file content or blindly from the extension?

### 2. Injection

- **Command injection**: any use of `exec`, `spawn`, `execFile`, or template strings passed to a shell — can attacker-controlled input reach these calls?
- **Path injection**: user input concatenated into file paths without normalisation
- **Template/expression injection**: any `eval`, `Function()`, or server-side template rendering with unescaped user data
- **Header injection**: user input reflected in HTTP response headers without sanitisation (CRLF injection)

### 3. Cross-Site Scripting (XSS)

- Is user input (URL params, cookies, headers) reflected in HTML responses without escaping?
- Are there any uses of `{@html ...}` (Svelte), `innerHTML`, or `document.write` with unsanitised data?
- Does the Content-Security-Policy header restrict inline scripts and untrusted origins?
- Are `<script>` tags dynamically generated with user-controlled content?

### 4. Access control & authorisation

- Are there routes or API endpoints that expose sensitive data without authentication?
- Can a user access another user's resources by manipulating slugs, IDs, or path segments?
- Is there any admin functionality that lacks an access check?
- Are download endpoints (CBZ, EPUB, MOBI) gated appropriately?

### 5. Sensitive data exposure

- Are secret keys, tokens, or credentials present in source files, config files, or `.env` files committed to the repo?
- Are internal stack traces or error messages returned to the client in production?
- Are server-side configuration values (paths, keys) accidentally exposed in client-side bundles or SSR responses?
- Does the `robots.txt` inadvertently reveal sensitive endpoint paths?

### 6. Security headers

Check the HTTP responses (look at server hooks, middleware, or framework config) for the following headers:

| Header | Expected |
|--------|----------|
| `Content-Security-Policy` | Defined and restrictive |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` or stricter |
| `Strict-Transport-Security` | Present if HTTPS is used |
| `Permissions-Policy` | Defined |

Are any of these missing or set to permissive values?

### 7. Cookie security

- Are session or tracking cookies missing `HttpOnly`, `Secure`, or `SameSite` attributes?
- Is the `vw`/`vh` cookie (viewport width) validated server-side before being used to build file paths or query params?
- Can an attacker poison a cookie to influence server behaviour in an exploitable way?

### 8. Dependency vulnerabilities

- Look at `package.json` for notably old or known-vulnerable packages
- Note any packages with a history of critical CVEs (e.g. prototype pollution, RCE)
- Do not run `npm audit` — analyse statically

### 9. Server-Side Request Forgery (SSRF)

- Does any route fetch a URL derived from user input (e.g. a cover URL, feed URL, external image proxy)?
- Could an attacker make the server send requests to internal services (localhost, metadata endpoints)?

### 10. Denial of Service (application-level)

- Are there endpoints that trigger expensive operations (resize, archive generation, scan) without rate limiting or size caps?
- Can an attacker cause unbounded memory usage by sending large inputs or triggering many concurrent operations?
- Is the in-memory cache (page cache, dimensions cache) bounded? Can it be exhausted?

### 11. Information disclosure via error handling

- Do catch blocks or error handlers return raw exception messages, stack traces, or internal paths to the client?
- Are 404 vs 403 responses distinguishable in a way that reveals the existence of restricted resources?

### 12. Dependency on environment / configuration

- What happens if `library_path` or `output_path` is set to a sensitive system directory?
- Is there any validation of config values that could lead to unexpected behaviour or privilege escalation?

## Severity scale

Assign each finding one of the following severities:

| Severity | Meaning |
|----------|---------|
| **CRITICAL** | Directly exploitable, leads to RCE, data exfiltration, or full access bypass |
| **HIGH** | Exploitable with moderate effort, significant data or functionality impact |
| **MEDIUM** | Exploitable under specific conditions, limited impact |
| **LOW** | Defence-in-depth issue, unlikely to be exploited in isolation |
| **INFO** | Hardening recommendation, not a vulnerability |

## Report format

Each finding must have a unique number (S-1, S-2, …) so the user can reference it directly.

```
# Web Security Audit

## Summary
[2-3 sentence synthesis: overall posture, most critical areas]

## Attack surface
[Brief description of exposed routes, file-serving logic, and external inputs]

## Findings

### S-1 — [Short title] [CRITICAL/HIGH/MEDIUM/LOW/INFO]
**Category**: Path Traversal / XSS / Injection / ...
**Location**: `file:line`
**Description**: What the vulnerability is and how it can be exploited.
**Proof of concept**: Concrete example — URL, payload, or code snippet that demonstrates the issue.
**Remediation**: Specific fix with code example if applicable.

### S-2 — ...

## Security headers audit
| Header | Status | Notes |
|--------|--------|-------|
| Content-Security-Policy | MISSING / PRESENT | ... |
| X-Content-Type-Options | MISSING / PRESENT | ... |
| X-Frame-Options | MISSING / PRESENT | ... |
| Referrer-Policy | MISSING / PRESENT | ... |
| Strict-Transport-Security | MISSING / PRESENT | ... |
| Permissions-Policy | MISSING / PRESENT | ... |

## Priority fixes

1. S-X [CRITICAL]: [One line — what to fix]
2. S-Y [HIGH]: [One line — what to fix]
...

## Out of scope / not analysed
[List any areas you could not assess due to missing context (e.g. runtime configuration, network layer)]
```

Be precise. Every finding must include a file path and line number. Avoid theoretical observations with no concrete exploit path — if you cannot show how it is exploitable, mark it INFO.

## Output

Once the report is complete, display it in the conversation, then ask the user if they want it saved to a file. If they confirm, save it to a file named `yyyy-mm-dd_hh-mm-ss_review-web-security.md` (using the current date and time) in a `reports/` directory at the root of the current working directory (create it if it does not exist).
