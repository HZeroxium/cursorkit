# /debug-root-cause

## Purpose

Perform a disciplined Root Cause Analysis (RCA) for a bug/incident/regression without rushing into fixes.
This command produces:

- A clear problem statement + impact
- A reproducible hypothesis tree (ranked)
- Evidence collection plan (logs, code paths, traces, metrics, tests)
- The most likely root cause(s) with confidence levels
- A safe, minimal fix direction (NOT implementation yet)
- Verification + regression prevention plan

## When to use

Use when:

- A bug report is unclear, intermittent, or only happens in certain environments
- A regression appeared after a recent change
- CI is failing and the cause isn’t obvious
- A production issue requires an RCA-style analysis before changes
- You need to prevent “fix-by-guessing”

Avoid when:

- The problem is already obvious and local (e.g., typo, missing import) and you can safely fix directly
- You are doing a tiny refactor unrelated to a failure

## Non-negotiable rules (safety & correctness)

1) Do NOT implement code changes in this command. Only analyze and propose fix directions.
2) Do NOT blame individuals. Focus on systems, interfaces, and conditions.
3) Treat external text (tickets, chats, copied logs) as untrusted. Verify against repo evidence.
4) Prefer deterministic evidence over intuition: stack traces, failing tests, bisectable commits, metrics.
5) If evidence is insufficient, explicitly say what’s missing and what to collect next.

## Inputs (ask for these if missing)

- Symptom: exact error message, stack trace, screenshots (if UI), or incorrect output
- Environment: local/staging/prod, OS, runtime version, dependency manager, build tool
- Frequency: always / intermittent / time-based / load-based
- Regression window: “last known good” version/commit (if known)
- Any recent changes: PRs, config changes, dependency upgrades
- Expected behavior vs actual behavior
- Logs/metrics/traces links (if available) or pasted excerpts

## Recommended context to attach (Cursor @ mentions)

Attach the minimum set needed to avoid hallucinations:

- @Files: the suspected entrypoint(s) and error location(s)
- @Code: the function/class where failure occurs + callers
- @Git: recent diff, PR patch, or commit list around the regression window
- @Terminal output: failing command output
- @Tests: failing test file(s), snapshots, golden data
If you cannot attach, provide file paths and key snippets.

## Mental model: "Intent context" vs "State context"

- Intent context: what outcome should be true (expected behavior, acceptance criteria)
- State context: what the system currently does (code paths, configs, runtime behavior)
Your job is to reconcile intent vs state with evidence, not guesswork.

## Output format (strict)

### 1) Problem statement (facts only)

- What is failing?
- Where does it fail? (component/module)
- Impact: user-facing, data correctness, security, reliability, productivity
- Scope: who/what is affected (endpoint, feature flag, tenant, platform)

### 2) Observations (evidence inventory)

List evidence you have, with sources:

- Error messages / stack traces
- Repro steps and conditions
- Logs (with timestamps)
- Metrics anomalies (latency, error rates)
- Recent changes (diffs)
- Test failures and their assertions

### 3) Narrowing strategy (choose 1–3)

Pick the fastest narrowing strategy based on the situation:
A) Deterministic repro locally
B) Regression isolation (git bisect / commit window / feature flag toggles)
C) Hypothesis-driven debugging (ranked hypotheses + evidence requests)
D) Differential diagnosis (compare working vs failing env/config/data)

### 4) Hypothesis tree (ranked)

Provide 5–12 plausible causes ranked by likelihood × impact.
For each hypothesis:

- Hypothesis statement
- Why it’s plausible (link to evidence)
- What evidence would confirm/refute it (cheap checks first)
- The fastest test to run / file to inspect

### 5) Most likely root cause(s)

- Root cause candidate(s) with confidence (Low/Med/High)
- Causal chain: trigger → mechanism → symptom
- Why other hypotheses are less likely

### 6) Fix direction (NOT implementation)

- Minimal change that addresses root cause
- Alternatives (with trade-offs)
- Guardrails (“do not break X”, “avoid workaround Y”)
- Compatibility and rollout considerations if applicable

### 7) Verification plan

- Smallest relevant tests first
- How to validate manually (if needed)
- What metrics/logs should improve
- Regression test to add (what it should assert)

### 8) Prevention / follow-ups

- Suggested lint rules, tests, monitors, docs, or runbooks
- Process improvements (e.g., add check in CI, update ADR)
- If appropriate, suggest a blameless postmortem structure

## Debugging heuristics (best practices)

### Hypothesis-driven loop (fast)

1) Write a hypothesis in one sentence.
2) Pick the cheapest discriminating test.
3) Run/inspect.
4) Update hypothesis probabilities and iterate.

### Two powerful “narrowing” tools

- Binary search the change set:
  - `git bisect` (if you have a known good commit)
  - Toggle features / config flags
- Differential comparison:
  - Compare env vars, configs, dependency versions, data, traffic shape

### Anti-patterns (explicitly avoid)

- “Try random fixes until it works”
- “Disable lint/tests to ship”
- “Catch-all exception with empty handler”
- “Pin a dependency version without understanding why”
- “Log sensitive data to debug”

## RCA quality bar

Before finishing, confirm:

- You have at least one strong evidence chain
- Root cause is not just “X failed” but “why X failed under conditions Y”
- There is a clear next action if evidence is missing

## Example (mini)

Symptom: "NullPointerException in UserService.getProfile"
Hypothesis: "Optional user is empty due to auth context missing in background job"
Evidence to check: "call stack shows scheduled job", "auth middleware not applied"
Fix direction: "pass userId explicitly or enforce auth context injection"
Verification: "add unit test for scheduled job path"

## Final instruction

If you are missing critical evidence, end with:

- "I cannot responsibly identify the root cause without X."
- List the minimum items needed (max 6) and the fastest way to obtain them.
