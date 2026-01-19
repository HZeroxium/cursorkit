# /risk-and-constraints

## Purpose

Produce a comprehensive risk and constraints assessment to prevent:

- Hidden security/privacy issues
- Performance regressions
- Breaking changes
- Operational surprises (rollout/rollback)
- “We shipped but can’t verify it” problems

This command outputs:

1) Constraints catalog (what must be true)
2) Risk register (what could go wrong)
3) “What NOT to do” guardrails
4) Mitigation and verification plan

## When to use

Use when:

- Changes touch auth, user data, billing, payments, permissions, or secrets
- Performance or reliability matters
- You are migrating dependencies, APIs, schemas, or infra
- The request mentions “optimize”, “secure”, “refactor”, “rewrite”, “upgrade”
- You’re uncertain about blast radius

## Required inputs

Ask for missing info:

- What environment(s) matter (local/staging/prod)?
- Any compliance constraints (PII, GDPR-like, internal policy)?
- Performance SLOs (latency, throughput)?
- Compatibility requirements (public API stability, DB schema stability)?
- Rollout approach preference (flag, canary, big-bang)?
- Time constraints / deadlines?

## Context to attach

Request:

- Relevant code paths and entrypoints (routes/services/jobs)
- Current architecture diagram or README (if exists)
- Logs/metrics dashboards, if any
- CI pipeline constraints (time limits, required checks)
- Dependency lockfiles, runtime versions

## Output format (strict)

### 1) Constraints catalog (grouped)

For each constraint:

- Constraint
- Why it exists
- How to verify it
- Who owns it (if known)

Groups to include:
A) Security & privacy

- Secrets management, token handling, storage of credentials
- AuthN/AuthZ invariants (permissions, roles, scopes)
- Input validation and injection vectors
- Logging redaction and PII exposure
- Supply chain constraints (dependency policies)

B) Reliability & correctness

- Idempotency, retries, timeouts
- Error handling guarantees
- Data consistency and transaction boundaries
- Concurrency constraints (race conditions, locks)

C) Performance & scalability

- Latency budgets (p50/p95/p99)
- Memory/CPU budgets
- Hot path constraints
- N+1 queries, caching strategy, payload sizes

D) Backward compatibility

- API contract stability
- DB schema migration safety (expand/contract)
- Versioning policy
- Deprecation strategy

E) Operability

- Observability requirements (logs/metrics/traces)
- Alerting/monitoring expectations
- Rollout/rollback requirements
- Runbooks and incident response

F) Delivery constraints

- Deadline, team capacity, risk tolerance
- Tooling constraints (lint, typecheck, CI gates)

### 2) Risk register (ranked)

For each risk:

- Risk description
- Likelihood (Low/Med/High)
- Impact (Low/Med/High)
- Detection method (how we’ll notice)
- Mitigation (prevent)
- Contingency (respond/rollback)

### 3) “What NOT to do” (explicit guardrails)

Write 10–20 bullets, tailored to the task.
Examples:

- Do NOT log raw tokens, passwords, or full request bodies containing PII.
- Do NOT change public API response shape without a versioning plan.
- Do NOT introduce a new dependency without checking license/vulnerability posture.
- Do NOT widen permissions to “fix” an auth issue; preserve least privilege.
- Do NOT ship without a way to verify success in staging (tests or metrics).

### 4) Recommended approach (constraints-aware)

- The safest viable approach that satisfies constraints
- Alternatives and trade-offs (at least 2 if meaningful)
- What you would choose and why

### 5) Verification plan

- Tests to run (smallest first)
- Metrics/log checks
- Manual validation steps
- Rollout validation gates (what must look good before proceeding)

### 6) Questions / missing info

If anything blocks a confident assessment, ask targeted questions (max 8).

## Guidance for reasoning (internal)

- Assume risk until proven otherwise on auth/data/secrets.
- Prefer reversible changes and feature flags.
- Favor backward-compatible migrations.
- Prefer “add then switch then remove” over “replace in one step.”

## Final instruction

End with:

- Top 3 risks
- Top 3 constraints
- The single most important verification gate
- A clear “stop” condition (when not to proceed)
