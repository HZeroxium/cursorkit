# /security-audit — Audit input validation, authn/authz, injection risks, secrets leakage, and risky operations

## Intent

You will perform a practical security audit that produces:

- Findings with severity and evidence
- Fix recommendations prioritized by risk reduction
- Concrete code changes only when safe and explicitly requested
- Guardrails to prevent recurrence (tests, lint rules, CI checks)

## When to use

- Before a release / security review.
- After incidents (token leak, injection, auth bypass).
- When adding new endpoints, file uploads, webhooks, or admin features.
- When integrating third-party SDKs or handling sensitive data.

## Required context (attach before running)

Attach:

- Entry points: routes/controllers/resolvers/RPC definitions.
- Auth setup: middleware/guards, session/JWT handling.
- Validation layer: schemas, DTOs, request parsing.
- Data sinks: DB queries, ORM usage, templating, shell exec, file IO.
- Secrets handling: env vars, config files, CI/CD pipelines.
- Threat model constraints: what assets are sensitive?

If missing, request:

- "Where is auth enforced?"
- "Where is input validation defined?"
- "Where are database queries and external calls performed?"

## Inputs (fill in)

- App type: [API / Web / Mobile backend / CLI]
- Auth type: [JWT/OAuth2/Session/API key]
- Data sensitivity: [LOW/MED/HIGH]
- Compliance constraints: [PII/PCI/PHI]
- Backward compatibility: [STRICT/RELAXED]
- Timebox: [TIMEBOX]

## Output artifacts

1) Findings report (ranked):
   - Title, severity, affected area, evidence, exploit scenario
2) Fix plan:
   - Immediate fixes (high severity)
   - Medium-term refactors
   - Guardrails (tests, CI checks)
3) Optional code changes (small diffs) if requested
4) Verification checklist

## Severity rubric (simple)

- Critical: auth bypass, RCE, secrets leak, SQL injection with exfiltration
- High: SSRF, IDOR, stored XSS, privilege escalation
- Medium: weak logging/monitoring, missing rate limits, verbose errors
- Low: hardening, headers, minor misconfigs

## Guardrails (do NOT do)

- Do NOT request or log secrets.
- Do NOT suggest insecure “quick hacks” (disable TLS verification, skip auth).
- Do NOT change auth flows without explicit approval.
- Do NOT remove validation to “fix bugs”.

---

## Audit checklist (prioritized)

### 1) Authentication (AuthN)

Check:

- Is authentication required for protected routes?
- Are tokens validated properly (signature, issuer, audience, expiry)?
- Are session cookies secure (HttpOnly, Secure, SameSite) if applicable?
- Are password flows safe (rate limiting, lockout, secure reset)?

Findings examples:

- Accepting unsigned JWT
- Missing expiry checks
- Session fixation risks

### 2) Authorization (AuthZ)

Check:

- Enforce resource-level authorization (RBAC/ABAC)
- Prevent IDOR:
  - user can access other user’s resources by guessing IDs
- Ensure admin paths are gated server-side (not just UI)

### 3) Input validation and parsing

Check:

- Validate and normalize inputs at boundaries
- Enforce:
  - types, ranges, lengths
  - allowed enums
  - strict schema parsing
- Reject unknown fields if your contract requires it

### 4) Injection risks

Check sinks:

- SQL/NoSQL queries (string concatenation?)
- Template injection
- Command injection (shell exec)
- Path traversal in file access
- SSRF in URL fetchers/webhooks

Rules:

- Use parameterized queries
- Allowlist hosts for outbound fetch
- Normalize paths and prevent `../`

### 5) Secrets handling and leakage

Check:

- Secrets in repo (config files, history)
- Secrets in logs (headers, payloads)
- CI/CD variables exposure
- Third-party keys in frontend bundles

Add guardrails:

- secret scanning
- pre-commit hooks
- log redaction middleware

### 6) Error handling & logging

Check:

- No stack traces to clients
- Logging exists for security events:
  - login failures
  - privilege changes
  - suspicious requests
- Protect logs from injection and avoid sensitive fields

### 7) Rate limiting and abuse protections

Check:

- Rate limits on auth endpoints, expensive endpoints
- Request size limits
- Pagination limits
- Anti-automation (where relevant)

### 8) Dependency and supply chain

Check:

- lockfiles present
- dependency updates policy
- known vulnerabilities scanning (SCA)
- avoid unpinned dependencies in prod

---

## Step-by-step workflow

Step 1: Enumerate entrypoints and trust boundaries.
Step 2: Identify sensitive assets and threat scenarios.
Step 3: Run checklist (AuthN/AuthZ/Validation/Injection/Secrets/Logging/Rate limits/Supply chain).
Step 4: Write findings with evidence:

- file:line references
- example exploit request (safe description)
Step 5: Propose fixes in priority order.
Step 6: Add regression tests/guardrails:
- authorization tests
- validation tests
- lint rules and CI checks

---

## Definition of Done

- [ ] Findings ranked with evidence
- [ ] Fix plan prioritized with owners and scope
- [ ] High severity issues have clear remediation steps
- [ ] Guardrails proposed to prevent recurrence
- [ ] Verification checklist exists (how to confirm fix)

## Example invocation

"Audit the user profile and admin endpoints for authz (IDOR), validate input schemas, check for SQL injection, secrets leakage in logs, and propose prioritized fixes + guardrails."
