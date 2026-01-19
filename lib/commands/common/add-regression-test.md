# /add-regression-test

## Purpose

Add a regression test that "locks in" a bug fix so the bug cannot silently return.
This command ensures the new test:

- fails before the fix (or would have failed),
- passes after the fix,
- is deterministic, isolated, and maintainable,
- lives at the correct layer (unit / integration / e2e),
- documents the bug scenario clearly.

A regression test is not "more tests".
It's a test that captures the specific failure mode and prevents recurrence.

---

## Inputs (ask if missing)

- Bug summary in 1 sentence
- Root cause summary (or best evidence-based hypothesis)
- Fix summary (what changed)
- Reproduction artifact:
  - failing request payload, minimal repro, stack trace, or failing scenario
- Constraints:
  - speed requirements (fast feedback vs full system)
  - permitted test types in repo (unit-only? integration ok?)
  - CI runtime budget

---

## Attach context (Cursor @ mentions)

Attach:

- @Files for the code under test and the exact fix diff
- @Files for existing test folders and helpers (to match conventions)
- @Configs for test runner and environment (if needed)
- @Docs if the project has testing standards or guidelines

---

## Decide test level (choose the lowest-cost layer that proves the bug)

Use this decision tree:

### 1) Can we reproduce with a pure function / class method?

→ Write a unit test.

### 2) Does it require a boundary (DB, filesystem, network abstraction, HTTP handler)?

→ Write a medium/integration test against an in-memory or hermetic dependency.

### 3) Does the bug only manifest via full workflow (UI + backend + async jobs)?

→ Write a small number of e2e tests for critical flows only.
Keep them stable; avoid flaky coverage.

---

## Regression test design rules (non-negotiable)

1) The test must be deterministic:
   - no reliance on real time unless controlled
   - no reliance on randomness unless seeded
   - no reliance on external networks
2) The test must be isolated:
   - cleanup after itself
   - no order dependence
3) The test must be specific:
   - assert the bug's failure mode
   - avoid overly broad assertions (“everything equals snapshot”)
4) The test must be readable:
   - encode "Given / When / Then" in structure and naming
5) Avoid accidental coupling:
   - do not assert on incidental implementation details unless necessary

---

## “Characterization test” option (when legacy code is hard to unit test)

If the area is high-risk legacy code with unclear behavior:

- Write a characterization test capturing current behavior first.
- Then refactor safely with that test as a safety net.
- Later, replace with intention-revealing tests if possible.

Use characterization tests intentionally; document why.

---

## Work plan (step-by-step)

### Step 1 — Define the bug contract

Write the contract in plain English:

- Given (preconditions)
- When (action)
- Then (expected outcome)
Include:
- the previous incorrect outcome (what used to happen)
- the correct outcome (what should happen now)

### Step 2 — Create a minimal fixture

- minimal input payload
- minimal DB rows (if needed)
- minimal mocked response (if needed)
Use stable, small fixtures.
Avoid production dumps and PII.

### Step 3 — Choose assertion strategy

Pick the most direct assertion:

- error type and message (if stable)
- status code + structured error body
- output value / invariants
- side effects:
  - DB row created/updated
  - event emitted
  - cache entry invalidated

Avoid:

- asserting on ordering unless relevant
- asserting on timestamps unless controlled
- giant snapshots as the only assertion

### Step 4 — Ensure it fails on old behavior

Options:

- If you can, run test against pre-fix commit (ideal).
- If not, provide a written proof:
  - explain why it would fail pre-fix (link to root cause)
  - optionally show a minimal reproduction snippet

### Step 5 — Implement the test

- Follow repo conventions for:
  - folder structure
  - naming
  - test helpers
  - setup/teardown
- Keep the diff small and focused on test addition.

### Step 6 — Verify determinism

- Run test multiple times locally (bounded).
- Run in CI-like mode if feasible.
- If flaky appears, treat as test defect and fix design.

### Step 7 — Document the intent

In the test, add:

- a concise comment referencing the bug scenario (not an external link unless allowed)
- why this case matters
- what invariants are being protected

---

## Output format (strict)

### 1) Regression test spec

- Bug contract: Given / When / Then
- Test level chosen and why
- Minimal fixture plan

### 2) Files to change

- path to new/updated test
- any helper/fixture files
- (optional) small harness or factory

### 3) Proposed test code outline

Provide a skeleton structure (language/framework-specific patterns if known):

- Arrange / Act / Assert
- setup/teardown
- assertions

### 4) Verification commands

- run single test
- run targeted suite
- optional full suite

### 5) DoD

- deterministic, isolated
- fails pre-fix (or convincingly would fail)
- passes post-fix
- understandable and minimal

---

## Common pitfalls (explicit "NO")

- NO: "test the fix" by copying production logic into the test
- NO: blanket snapshot updates as a regression lock
- NO: relying on real external services
- NO: sleeps instead of explicit waits
- NO: asserting on volatile values (timestamps, random ids) without control

---

## Final instruction

If the repository’s testing framework is unknown, first inspect existing tests to mirror patterns.
Do not introduce new testing libraries unless explicitly requested.
