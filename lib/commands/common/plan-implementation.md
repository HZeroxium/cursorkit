# /plan-implementation

## Purpose

Create a safe, reviewable, step-by-step implementation plan with:

- File impact map (what files change and why)
- Milestones + checkpoints
- Test strategy (smallest relevant tests first)
- Rollback plan and operational considerations
- Clear Definition of Done (DoD)

This command is intentionally “Plan first, code later.”

## When to use

Use when:

- The change touches multiple files/modules
- You are refactoring, migrating, or changing behavior
- You need coordination across API/DB/UI layers
- You want to reduce risk and avoid scope creep

## Preconditions

- Requirements are clarified and acceptance criteria exist.
- If requirements are unclear, stop and run /clarify-requirements first.

## Ground rules

1) Do not implement. Output only a plan.
2) Prefer small diffs and incremental checkpoints.
3) Identify the smallest test loop first (fast feedback).
4) Surface risks early; propose mitigations.
5) If you need more context, ask targeted questions before finalizing the plan.

## Context gathering steps (do this first)

1) Identify relevant entrypoints and call paths:
   - Where does the flow start? (route/controller/CLI/main)
   - Which modules own the behavior?
2) Identify dependencies and constraints:
   - external services, DB schema, config flags, auth, caching
3) Identify existing tests and how to run them:
   - unit/integration/e2e
4) Identify observability hooks:
   - logs/metrics/traces relevant to verifying behavior
5) Summarize “current state” vs “desired state” in 3–6 bullets.

## Output format (strict)

### 1) Plan overview (executive summary)

- Goal
- Scope
- Non-goals
- High-level approach (3–7 bullets)

### 2) File impact map

List files grouped by intent. For each file:

- Path
- Why it changes
- Type of change (add/modify/delete)
- Risk level (Low/Med/High)
Example:
- `src/api/auth/login.ts` — modify — add validation and new error mapping — Med risk

### 3) Step-by-step implementation plan (with checkpoints)

Write numbered steps. Each step must include:

- Objective
- Concrete edits (what to change)
- Verification (what to run/check)
- Exit criteria (how to know the step is done)

Include explicit checkpoints such as:

- Checkpoint A: compile/typecheck passes
- Checkpoint B: unit tests pass
- Checkpoint C: integration scenario validated

### 4) Testing strategy (smallest relevant tests first)

- Smoke tests (fast)
- Unit tests to add/update
- Integration tests to add/update
- e2e tests (only if needed)
- “Golden” examples (if applicable)
Also include:
- How to validate locally
- How to validate in CI
- What metrics/logs to inspect

### 5) Risk register + mitigations

For each risk:

- Risk statement
- Likelihood (L/M/H)
- Impact (L/M/H)
- Mitigation plan
Examples:
- Backward compatibility break → add adapter layer, keep old API stable
- Performance regression → add benchmark or profiling step

### 6) Rollout & rollback plan (if applicable)

- Feature flag strategy
- Gradual rollout
- Backward-compatible DB migrations
- Rollback steps (including data considerations)

### 7) Definition of Done (DoD)

A checklist of “done means done”, e.g.:

- All acceptance criteria met
- Relevant tests added and passing
- Lint/typecheck/build passing
- No new security risks
- Docs updated if behavior changed
- Observability added/updated if needed

### 8) Open questions / dependencies

- Remaining unknowns that block implementation
- External dependencies (other teams, secrets, infra)

## Planning heuristics (best practices)

- Prefer “thin slice” end-to-end first if feature work (minimal happy path)
- Prefer “safety refactor” in separate commits from behavior changes
- If refactoring, keep each step behavior-preserving until final switch
- Always include a verification step per milestone

## Optional: Suggested work breakdown for commits

If appropriate, propose a commit sequence:

1) Add/adjust tests
2) Refactor to enable change (no behavior change)
3) Implement behavior change
4) Add observability/docs
5) Cleanup

## Final instruction

End your response by explicitly asking the user to approve the plan before you start implementing.
