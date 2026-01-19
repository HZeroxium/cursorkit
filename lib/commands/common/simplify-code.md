# /simplify-code

## Purpose ("Deslop")

Simplify code to reduce complexity and duplication while improving readability and maintainability,
WITHOUT changing behavior unless explicitly allowed.

This command targets:

- unnecessary complexity,
- duplicated logic,
- unclear naming and structure,
- high-cognitive-load code paths,
- accidental complexity (extra abstractions that don’t pay off).

It is not about personal style. It is about making future changes safer and faster.

---

## Inputs (ask if missing)

- The target area:
  - file(s) / module(s) / function(s)
- The motivation:
  - hard to read?
  - too many bugs?
  - too many branches / conditions?
  - duplication across files?
  - performance concerns?
- Constraints:
  - behavior must stay identical? (default yes)
  - public API stable? (default yes)
  - time budget / scope budget

---

## Attach context (Cursor @ mentions)

Attach:

- @Files for the target code
- @Files for representative callers
- @Tests for coverage around the target
- @Docs for project conventions and architecture boundaries

---

## Simplification principles (high leverage)

### 1) Make the code say what it means

- Rename for intent (domain language)
- Prefer explicitness over cleverness
- Avoid “action at a distance”

### 2) Reduce branching and nesting

- Replace nested ifs with:
  - guard clauses
  - early returns
  - extracted predicate functions
- Flatten control flow where safe.

### 3) Remove duplication

- Extract shared logic into:
  - a helper function
  - a small class with a single responsibility
- But avoid “premature abstraction”:
  - if only used once, keep it local unless it improves clarity.

### 4) Make data flow obvious

- Limit mutation and hidden side effects
- Prefer immutable data transforms (where idiomatic)
- Ensure inputs/outputs are clearly defined

### 5) Keep responsibilities small

- Split god functions/classes into cohesive units
- Ensure each unit has a clear contract

---

## Practical workflow (step-by-step)

### Step 0 — Lock behavior

- Identify the contract of the code:
  - inputs, outputs, side effects
- Ensure tests exist:
  - if tests are weak, add characterization tests first.

### Step 1 — Identify code smells (symptoms of deeper issues)

Look for:

- Duplicated code
- Long methods / long parameter lists
- Large classes / “god objects”
- Too many conditionals / switch cases
- Magic numbers/strings
- Unclear naming
- Side-effect-heavy functions
- Implicit dependencies (globals, env reads in deep layers)

Write a short list of the top 3–7 smells with file/line references.

### Step 2 — Pick simplification moves (ranked by safety)

Start with the lowest-risk moves:

1) Rename identifiers for clarity (no semantic change)
2) Extract function for repeated blocks
3) Replace complex expressions with named variables
4) Introduce guard clauses to reduce nesting
5) Normalize data shapes at boundaries
6) Split responsibilities (extract class/module)
7) Remove dead code / unreachable branches

### Step 3 — Apply changes in small patches

- After each move, run the smallest relevant tests.
- Keep diffs small and readable.
- Avoid mixing formatting-only changes unless required.

### Step 4 — Add clarity scaffolding (optional)

If the logic is inherently complex:

- add comments explaining “why”, not “what”
- add small diagrams in code comments (e.g., state machine states)
- add input validation at boundaries
- add explicit error types/messages (if consistent with repo)

### Step 5 — Validate no behavior changes

Verification options:

- existing unit/integration tests
- new regression tests for tricky cases
- golden tests for stable fixtures
- manual smoke checks if needed

---

## Output format (deliverable)

### 1) Simplification goals

- What will be simpler after this?
- What will remain unchanged?

### 2) Smell inventory (top findings)

- Smell: description
- Location: file path + function name
- Why it hurts: readability, bugs, change risk

### 3) Proposed plan (with file impact map)

- Step-by-step list
- Which files change at each step
- Checkpoints (tests/commands)

### 4) Proposed patch strategy

- Minimal diff approach
- Commit splitting plan (if significant)

### 5) Risk & guardrails

- What could accidentally change behavior?
- How we prevent it (tests, invariants, assertions)

### 6) Verification commands

- smallest loop
- expanded loop

---

## Examples of “good simplification” outcomes

- A 60-line nested conditional becomes:
  - 5 guard clauses + 3 extracted helpers
  - each helper named after domain intent
- 3 duplicated validation blocks become:
  - a shared validator with clear contract
- A function with 9 parameters becomes:
  - a value object / config object with explicit fields
- A “do everything” service becomes:
  - small components with clear responsibilities and tests

---

## Anti-patterns (explicit "NO")

- NO: introducing layers/abstractions that add ceremony without benefit
- NO: “DRY at all costs” causing indirection everywhere
- NO: changing behavior under the guise of “cleanup” without approval
- NO: deleting tests to simplify changes
- NO: ignoring performance implications on hot paths

---

## Definition of Done (DoD)

- Complexity and duplication measurably reduced (by code review judgment or simple metrics).
- Readability improved: code communicates intent.
- Behavior unchanged (or changes explicitly documented and approved).
- Tests remain green and deterministic.
- Diff remains reviewable; commits are clean.

---

## Final instruction

Prefer the smallest set of changes that achieves the simplification goal.
If the code is under-tested, propose characterization/regression tests before deep simplification.
