# /test-plan

## Purpose

Create a fast, risk-driven testing plan for a change (feature / bugfix / refactor / dependency update).
This is not a generic checklist. It is a prioritized plan that:

- targets the highest-risk behaviors first,
- chooses the right test levels (unit/integration/e2e),
- includes edge cases and failure modes,
- defines smoke checks for rollout,
- includes rollback readiness.

The goal is: high confidence with minimal wasted effort.

---

## Inputs (ask if missing)

- Change summary (what changed, where)
- Risk profile: data correctness, security, availability, performance, backward compatibility
- Deployment model: library / service / web app / mobile / monolith / microservices
- Critical user flows affected (top 1–5)
- Existing tests: where they live, how long they take, what CI runs

---

## Attach context (Cursor @ mentions)

- @Files for main change diff (or list of changed files)
- @Files for test directories near the change
- @CI config (optional) if pipeline constraints matter
- @Docs / team rules about testing standards (if any)

---

## Risk-first planning model

Think in terms of "what can go wrong" rather than "what can we test".
Use the following risk buckets:

1) Correctness risks

- incorrect business logic
- wrong output formats
- inconsistent validation

1) Reliability risks

- timeouts, retries, race conditions
- resource leaks
- partial failures

1) Security risks

- authz/authn bypass
- injection and unsafe parsing
- sensitive data leakage in logs/errors

1) Compatibility risks

- API contracts (requests/responses)
- DB schema changes
- client compatibility / version skew

1) Performance risks

- hot paths slower
- increased memory/cpu
- N+1 queries or excessive calls

---

## Output format (deliverable)

### A) Test objectives (1–2 paragraphs)

- What confidence are we aiming for?
- What is explicitly out of scope?

### B) Change surface map (small)

List:

- entrypoints (API endpoints, UI routes, CLI commands)
- data stores touched
- external dependencies touched
- background jobs/events touched

### C) Prioritized test matrix (copy-paste friendly)

Use this table format:

| Priority | Area / Flow | Risk | Test level | Cases to cover | How to run | Owner / Notes |
|---------:|-------------|------|------------|----------------|-----------|--------------|
| P0       |             |      |            |                |           |              |

Guidance:

- P0: must run before merging
- P1: should run before release / same day
- P2: can run later / nightly
Keep the table small (typically 8–20 rows).

### D) Test cases (structured)

For each P0/P1 item, write a mini spec:

**Case name**

- Given:
- When:
- Then:
- Edge cases:
- Negative cases:
- Observability: logs/metrics to check (if relevant)

### E) Smoke checks (post-deploy)

Define the smallest set of manual/automated checks to validate in production/staging:

- 3–10 checks max
- focus on critical flows
- include a “canary” if possible

### F) Rollback plan

- What signals trigger rollback?
- How to rollback (feature flag, revert, config change)
- Data considerations (migrations, queued jobs, caches)

### G) Test data & environments

- Required fixtures / seeded data
- Hermetic/local options (containers, in-memory deps)
- If env-sensitive: record exact versions

### H) Exit criteria (DoD for testing)

- what must pass before merge
- what must pass before release
- who signs off (if applicable)

---

## Strategy recommendations (default)

### Prefer smaller & more deterministic tests

- Put most confidence in fast, deterministic tests.
- Use larger/e2e tests sparingly for critical flows only.
- Avoid creating a large flaky e2e suite.

### Avoid “false confidence”

- Do not test by replicating production logic in the test.
- Do not assert on incidental formatting if not part of the contract.

---

## Special sections (only include if relevant)

### If your change touches concurrency/async

- add stress-ish tests with bounded loops
- assert idempotency or ordering invariants
- ensure explicit waits (no sleeps)

### If your change touches time

- freeze time / inject clock
- avoid real-time dependence

### If your change touches external services

- contract tests or stable mocks
- retry/backoff policies verified
- error mapping tested

### If your change touches DB

- migration tests
- transaction boundaries
- backward compatible reads/writes

---

## Final instruction

Produce the full test plan with a prioritized matrix.
Then recommend the single fastest “minimal confidence loop” (a short command list) to run before committing.
