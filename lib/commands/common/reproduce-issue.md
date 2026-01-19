# /reproduce-issue

## Purpose

Convert a bug report into a reproducible scenario that developers can run reliably.
This command outputs:

- Clear reproduction steps
- A minimal reproducible example (MRE) plan (or actual minimal code sketch if safe)
- Expected vs actual behavior
- Environment matrix (where it repros / doesn’t)
- A reproduction artifact checklist (scripts, fixtures, sample inputs)
- “Done when repro is stable” criteria

## When to use

Use when:

- A bug can’t be reproduced locally
- Reports are inconsistent across machines/environments
- The bug is intermittent and needs stabilization
- You need a deterministic failing test or script before fixing

Avoid when:

- The bug already has a stable failing test case
- The request is “just explain what’s wrong”, not reproduce

## Non-negotiable rules

1) Prefer minimal, deterministic reproduction over broad “run the whole app and click around”.
2) If the issue is intermittent, capture timing/load/seed and aim for reproducibility ≥ 80%.
3) Never require secrets or PII in reproduction artifacts. Use sanitized fixtures.
4) Do not implement the full fix here; reproduction comes first.

## Inputs (ask if missing)

- Bug description in one sentence
- Expected behavior (what should happen)
- Actual behavior (what happens)
- Error messages, screenshots, stack traces
- Environment details:
  - OS, runtime versions (Node/Python/JDK), package manager
  - DB/cache/external services used
  - Feature flags or config toggles
- “Where it happens”: endpoint/screen/command
- Any sample payloads or files (sanitized)

## Context to attach (Cursor @ mentions)

- @Files: the affected component(s), suspected entrypoint(s)
- @Terminal: failing commands output
- @Git: recent commits if regression
- @Tests: existing tests near the behavior
- @Configs: env example file, docker compose, CI pipeline snippet

## Output format (strict)

### 1) Issue summary (concise)

- One-sentence description
- Severity and impact (user/data/security/availability/dev productivity)

### 2) Expected vs Actual

**Expected**

- Bullet list, include response codes, UI states, output formats

**Actual**

- Bullet list, include exact error messages or incorrect outputs

### 3) Reproduction steps (human steps)

Write steps that a new developer can follow:

- Pre-reqs (setup, dependencies, data)
- Commands to run (copy-paste)
- Actions to take (UI clicks, API calls, etc.)
- The exact moment the failure is observed
Include:
- "If you don’t see it, try X" (only if evidence-based)
- Timeouts/waits only if necessary, with reason

### 4) Minimal Repro Strategy (MRE plan)

Choose one or more approaches:
A) Minimal failing test (preferred)

- Identify the smallest unit/integration test that should fail
- Which assertion should fail and why
- How to run it

B) Minimal script / harness

- Single command script that triggers failure
- Inputs, outputs, exit codes

C) Minimal API call

- cURL request with payload
- Required headers (no secrets; use placeholders)
- Expected response vs actual response

D) Minimal UI scenario

- Use a stable route and minimal dataset
- Avoid multi-step flows unless necessary

### 5) Repro artifact checklist

List the files or snippets you will create/collect:

- `repro/README.md` (steps)
- `repro/request.json` (sample input)
- `repro/run.sh` or `repro/run.ps1` (script)
- `repro/fixtures/*` (sanitized data)
- `repro/Dockerfile` (optional, if env-sensitive)
- `repro/seed.txt` (random seed if relevant)

### 6) Environment matrix

A simple matrix:

- Works on: (env A, env B)
- Fails on: (env C)
- Unknown: (env D)
List key variables:
- runtime version, OS, container vs local, DB versions, config flags

### 7) Stabilization tactics (for flaky/intermittent issues)

Pick only relevant ones:

- Control randomness: fixed seed, deterministic ordering, stable clocks
- Control concurrency: single-thread mode, reduced parallelism
- Control timing: mocked time, reduced timeouts
- Control external dependencies: stub/mock, local container
- Add instrumentation: temporary logs with correlation IDs (no secrets)

### 8) “Repro is done when…”

Define success criteria:

- Repro steps run in ≤ N minutes
- Fails ≥ 4/5 times (or deterministic 100%)
- Minimal artifacts committed and documented
- No secrets/PII included

### 9) Next step handoff

- Once repro is stable, run /debug-root-cause or /fix-runtime-error or /fix-build as appropriate.

## Templates (copy-paste)

### Repro README skeleton

- Setup
- Steps
- Expected vs Actual
- Notes / env
- Troubleshooting

### Minimal cURL skeleton

curl -X POST "<https://HOST/path>" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <REDACTED>" \
  -d @repro/request.json

## Guardrails

- Don’t broaden scope to “rewrite tests” or “refactor modules”
- Don’t add a reproduction that requires production access
- Don’t embed real tokens in repro docs

## Final instruction

End with:

- The single best reproduction approach to try first
- The minimum context still needed (max 5 items)
- A clear command list a developer can copy-paste to attempt repro
