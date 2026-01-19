# /explain-changes

## Purpose

Explain a set of code changes (diffs) like a strong reviewer would:

- What changed (behavior, interfaces, data model)
- Why it changed (intent and trade-offs)
- How it works (key implementation notes)
- Risks and edge cases
- Verification steps (tests, commands, metrics)
- Suggested follow-ups (cleanup, docs, refactors)

## When to use

Use when:

- You have a PR/MR diff and want a high-quality explanation
- You want to prepare PR description, release notes, or reviewer guidance
- You want to detect risky changes before merging

## Inputs

Ask the user to provide at least one:

- `git diff` output, or
- PR link/text pasted (if available), or
- A list of changed files + selected code chunks
Also ask:
- The change goal / ticket summary
- Any constraints (backward compat, security, perf)

## Context checklist

Request/attach:

- Changed files
- Related tests
- CI output if it exists
- Logs or screenshots if UI changes
- Any impacted API contracts

## Output format (strict)

### 1) Summary (non-technical, 5 bullets max)

- What this change does from a user/system perspective

### 2) What changed (technical breakdown)

Group changes by category:

- Behavior changes
- API/interface changes
- Data model / persistence changes
- Config/infra changes
- Refactors (no behavior changes)
For each group:
- Reference specific files
- Highlight breaking vs non-breaking changes

### 3) Why this approach (trade-offs)

- Explain the design choice
- Alternatives considered (at least 1–2 if non-trivial)
- Why the chosen approach is reasonable under constraints

### 4) How it works (key mechanics)

Focus on:

- Control flow (where execution starts and ends)
- Data flow (inputs → transformations → outputs)
- Invariants (what must remain true)
- Error handling and retry logic
Keep it concise but precise; avoid reprinting whole files.

### 5) Risk assessment

List risks in descending priority:

- Security/privacy risks
- Correctness risks
- Performance risks
- Compatibility risks
- Operational risks
For each:
- Why it’s a risk
- How to detect it
- How to mitigate it (or what follow-up is needed)

### 6) Edge cases / failure modes

- List important edge cases this change introduces or modifies
- Expected behavior for each case

### 7) Verification checklist

Provide a runnable checklist:

- Local commands (smallest first)
- Specific tests to run
- What logs/metrics to inspect
- How to validate manually (if needed)

### 8) Review notes (what a reviewer should focus on)

- 5–12 bullets of “review hotspots” and questions
Examples:
- “Confirm this new validation doesn’t reject legitimate inputs”
- “Double-check that this migration is backward compatible”
- “Ensure secrets are not logged”

### 9) Suggested improvements / follow-ups

- Optional refactors
- Documentation updates
- Additional tests that would increase confidence
- Observability improvements

## Guardrails

- Do not claim behavior changes unless clearly supported by diff evidence.
- If intent is unclear, explicitly say “Intent unclear” and ask 1–3 clarifying questions.
- Avoid nitpicks; prioritize correctness, safety, maintainability, and readability.

## Final instruction

End with:

- “Merge readiness” verdict: (Ready / Needs changes / Blocked)
- Top 3 must-fix items (if any)
- Top 3 nice-to-haves (if any)
