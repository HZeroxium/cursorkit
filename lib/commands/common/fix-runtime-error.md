# /fix-runtime-error

## Purpose

Fix runtime errors (exceptions, null/undefined, edge cases, integration failures) with guardrails.
This command outputs:

- Root cause summary (or best evidence-based hypothesis)
- Minimal patch plan (or actual minimal patch if safe)
- Added/updated tests (regression-focused)
- Improved error handling, logging, and input validation (as needed)
- Verification steps and rollback considerations

## When to use

Use when:

- The app runs but fails at runtime: exceptions, incorrect behavior, 5xx, crashes
- Edge cases break: empty inputs, missing fields, concurrency issues
- Integration failures: timeouts, retries, external API changes
- A fix requires guardrails and regression tests

Avoid when:

- The failure is compile/build-time (use /fix-build)
- The failure is only “tests failing” without runtime evidence (use /fix-tests)
- Requirements are unclear (use /clarify-requirements)

## Preconditions

- You have at least one of:
  - stack trace, logs, failing request payload
  - stable repro steps
  - failing test that demonstrates the runtime issue

If none exist, first run /reproduce-issue.

## Non-negotiable rules

1) Fix the root cause, not the symptom. Avoid catch-all patches without explanation.
2) Add a regression test when possible.
3) Do not leak secrets/PII in logs or error messages.
4) Keep the diff small. If refactor is needed, split into safe commits.
5) Maintain backward compatibility unless explicitly approved.

## Inputs (ask if missing)

- Exact error and stack trace
- Affected endpoint/screen/command
- Expected vs actual behavior
- Sample input/payload (sanitized)
- Environment details (local/staging/prod)
- Constraints: performance budgets, backward compat, security requirements
- Whether retries/timeouts are involved

## Context to attach (Cursor @ mentions)

- @Files: the failing code path, entrypoint, and nearest callers
- @Logs: relevant log lines around the timestamp
- @Tests: existing test files near the behavior
- @Config: env vars and config files that affect the code path
- @Git diff: recent changes if regression

## Output format (strict)

### 1) Runtime failure summary

- Symptom (error message)
- Location (file/function/class)
- Trigger conditions (inputs, env, timing)
- Impact (user/data/reliability)

### 2) Root cause analysis (concise)

- Causal chain: trigger → mechanism → symptom
- Supporting evidence (logs/tests/code)
- If uncertain: top 2 hypotheses and what evidence would disambiguate

### 3) Fix strategy (minimal)

Choose one:
A) Validate inputs + fail fast with clear errors
B) Add null safety / optional handling / default behavior
C) Fix state management / concurrency / ordering
D) Fix integration boundary: retries/timeouts/backoff, schema changes
E) Fix resource lifecycle: connections, streams, file handles
For the chosen strategy:

- Exactly what to change
- Where to change it
- Why it’s minimal and safe

### 4) Guardrails & error model

- Ensure errors are:
  - actionable
  - not leaking sensitive info
  - consistent with repo conventions
- Decide:
  - when to return error vs fallback vs partial results
  - whether to retry or fail fast
- If external calls exist:
  - timeouts
  - retry policy (avoid infinite loops)
  - circuit breaker patterns if available
  - idempotency considerations

### 5) Regression tests

- What test to add/update
- What it asserts (Given/When/Then)
- Why it prevents recurrence
- How to run the smallest test loop

### 6) Observability upgrades (only if useful)

- Add logs at boundaries:
  - correlation IDs, request IDs
  - structured fields
- Add metrics/traces if available:
  - error rate counters
  - latency histograms
- Ensure redaction and safe logging

### 7) Verification checklist

- Minimal local reproduction
- Test commands (smallest first)
- Manual validation steps if needed
- CI checks

### 8) Rollback / safety plan (if risky)

- Feature flag (if available)
- Gradual rollout steps
- Rollback steps
- Data migration considerations (if touched)

## Implementation guidelines (best practices)

### Fastest safe loop

1) Reproduce the error (or failing test).
2) Implement minimal fix.
3) Add regression test.
4) Run smallest tests.
5) Expand checks if needed.
6) Confirm logs/metrics look sane.

### Anti-patterns

- Silent failure (catch and ignore)
- Over-broad try/catch that hides bugs
- “Fix” by increasing timeouts arbitrarily
- Removing validation instead of correcting logic
- Logging entire payloads containing PII/secrets

## Quality bar / DoD

- Root cause explained
- Regression test added or justified why not possible
- Fix is minimal and localized
- No sensitive data leaked
- All relevant tests/lint/build pass
- If behavior changes, docs updated

## Final instruction

End with:

- A patch-ready checklist of exact file edits (or the actual patch if approved)
- The smallest verification command set
- Any remaining open questions or risks
