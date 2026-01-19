# /refactor-safely

## Purpose

Perform a small, safe refactor that preserves behavior while improving structure.
This command enforces:

- behavior-preserving transformations,
- small diffs and checkpoints,
- tight feedback loops (tests stay green),
- commit discipline (separate refactor from behavior changes),
- explicit safety checklist.

Refactoring is not rewriting. It is controlled, incremental change.

---

## Inputs (ask if missing)

- Refactor goal (one sentence), e.g.:
  - reduce duplication in module X
  - extract function for readability
  - split a large class
  - improve naming and boundaries
- Scope constraints:
  - which folders/modules are in scope
  - time budget (small refactor only)
  - forbidden changes (public API, behavior, perf)
- Safety net:
  - existing tests?
  - can we add characterization tests first if coverage is poor?

---

## Attach context (Cursor @ mentions)

Attach:

- @Files: target code region and its callers
- @Files: existing tests covering the behavior (if any)
- @Git: recent diffs if this refactor is part of ongoing work
- @Docs: coding standards / architecture rules

---

## Refactor approach (two tracks)

### Track A — With good tests (preferred)

- Keep tests green after each micro-step.
- Use refactoring steps that are "too small to fail".

### Track B — With weak/no tests (legacy-safe mode)

- First add characterization tests that capture current behavior.
- Only then refactor in small steps.
- Prefer mechanical refactors with minimal semantic risk.

---

## Non-negotiable rules (behavior preservation)

1) Do NOT change external behavior unless explicitly approved.
2) Do NOT mix refactor with feature changes in the same commit/PR section.
3) Each step must be verifiable (tests or checks).
4) If a step cannot be verified, it is too large — split it.

---

## Step-by-step workflow (with checkpoints)

### Step 0 — Baseline and safety net

- Run the smallest relevant tests to establish a green baseline.
- Record:
  - commands
  - current behavior expectations
- If no tests exist:
  - add 1–3 characterization tests around the risky area first.

### Step 1 — Map dependencies and boundaries

- Identify:
  - public API surface
  - callers
  - side effects (IO, DB, network, global state)
- Mark "do not touch" boundaries.

### Step 2 — Choose refactoring moves (small & mechanical)

Pick only what fits the goal:

- Extract function / method
- Rename variables for clarity
- Inline variable/function (when it simplifies)
- Split long method into small cohesive units
- Remove duplication (extract shared helper)
- Replace conditional with polymorphism (only if low risk and well-tested)
- Move method to better module boundary (requires caller updates)

### Step 3 — Execute in micro-commits (recommended)

A typical safe sequence:

1) Pure renames (no behavior change) → run tests → commit
2) Extract helpers (no logic change) → run tests → commit
3) Remove duplication (redirect callers) → run tests → commit
4) Cleanup (dead code, unused imports) → run tests → commit

### Step 4 — Keep diffs reviewable

- Avoid “format-the-world”.
- No unrelated refactors.
- Ensure each commit message states the intent.

### Step 5 — Validate no behavior drift

Verification methods (choose appropriate):

- unit tests
- integration tests
- golden tests (stable fixtures)
- snapshot tests only if stable and reviewed
- manual smoke checks for critical flows

---

## Safety checklist (must include in output)

### Behavior safety

- [ ] Public API unchanged
- [ ] Exceptions and error codes unchanged (unless approved)
- [ ] Logging behavior acceptable (no secrets, no noise increase)
- [ ] Performance not obviously worse on hot paths

### Test safety

- [ ] Relevant tests exist and pass
- [ ] Added characterization tests if coverage was weak
- [ ] No new flakiness introduced (avoid sleeps)

### Change control

- [ ] Small commits
- [ ] Diff scoped to target area
- [ ] Rollback is easy (revert single commit)

---

## Output format (strict)

### 1) Refactor goal & scope

- Goal
- In-scope files
- Out-of-scope boundaries

### 2) Safety net assessment

- Existing tests coverage (what covers what)
- Whether characterization tests are needed

### 3) Plan with file impact map

- step-by-step plan
- exact files impacted each step
- checkpoints (what to run after each step)

### 4) Proposed commits (optional but recommended)

List commit titles in order.

### 5) Risks & mitigations

- what could break
- how we detect it early

### 6) Verification commands

- smallest loop
- expanded loop

---

## Anti-patterns ("NO")

- NO: “Big bang” refactor across many modules at once
- NO: behavior changes sneaked in as refactor
- NO: removing tests to make refactor easier
- NO: refactor without any safety net in high-risk code

---

## Definition of Done (DoD)

- Code is simpler/cleaner while behavior remains the same.
- Tests are green, and the change is easy to review.
- Any added tests are deterministic and minimal.
- Clear commit trail exists for easy revert.

---

## Final instruction

If you detect weak tests, pause and propose characterization tests first.
Otherwise proceed with micro-steps and checkpoints, keeping the suite green.
