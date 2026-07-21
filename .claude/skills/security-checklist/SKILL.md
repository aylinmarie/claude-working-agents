---
name: security-checklist
description: Canonical security checklist shared by the engineer, reviewer, and improver agents. Use when implementing code that touches user input, auth, secrets, or crypto; when reviewing a diff for security issues; or when scanning a codebase for security findings.
---

This is the single source of truth for security criteria across the engineer → tester → reviewer pipeline and the standalone improver scan. All three agents apply the same list below — only the framing of what to do with a hit differs by role (see "Applying this by role").

## Checklist

- Is user input sanitized before use in queries, commands, or output (SQL injection, command injection, XSS)?
- Are secrets, credentials, or tokens handled safely — not logged, not hardcoded, not exposed in client bundles?
- Are file paths validated to prevent path traversal (`../` sequences, symlink escapes)?
- Is authentication required on all routes/endpoints that need it? Can auth be bypassed?
- Are authorization checks present and correct — does the code verify the caller has permission, not just that they are authenticated?
- Is CSRF protection in place for state-mutating endpoints that accept cookies?
- Are new dependencies from trusted sources and pinned to exact versions? (For a scan, also run `npm audit` / `pip-audit` / `cargo audit` / `bundle audit` and report any HIGH or CRITICAL CVEs.)
- Do HTTP responses set appropriate security headers (CSP, X-Frame-Options, HSTS)?
- Is sensitive data (PII, financial, health) encrypted at rest and in transit?
- Are error messages sanitized to avoid leaking stack traces, internal paths, or schema details to end users?
- Are cryptographic operations using strong algorithms (no MD5/SHA1 for integrity, no ECB mode, no hardcoded IVs)?
- Is insecure deserialization or `eval()`/`innerHTML` on external input avoided?

## Applying this by role

**Engineer (building):** Treat this as a build-time constraint list. Never construct queries or shell commands from unsanitized input, never hardcode credentials, validate all inputs at system boundaries, avoid `eval()`/`innerHTML` with external data.

**Reviewer (auditing a diff):** Walk the checklist against the changed code. Any hit is a blocking finding — `[SECURITY] file:line — description` — regardless of perceived severity. Security findings are always blockers, never non-blocking observations.

**Improver (scanning a codebase):** This is Tier 1 of the scan, always run first, report every finding regardless of how minor. Use the standard finding format (Title / Risk / Suggestion / Effort).
