# /fix-build

## Purpose

Diagnose and fix build/compile failures in a clean, deterministic way, minimizing “dirty workarounds”.
This command can produce either:

- A fix plan (if the cause is unclear), or
- A minimal patch set (if the cause is clear and safe)
Always include:
- Root cause explanation
- Verification steps (local + CI)
- Guardrails to avoid breaking other platforms/envs

## When to use

Use when:

- Build fails locally or in CI (compile/typecheck/linking)
- Dependency resolution errors occur
- Toolchain mismatch errors (Node/Python/JDK, lockfiles)
- Lint/typecheck are wired into build and failing

Avoid when:

- The failure is runtime-only (use /fix-runtime-error)
- You’re hunting down test failures only (use /fix-tests)

## Inputs (ask if missing)

- The exact failing command (e.g., `npm run build`, `mvn test`, `gradle build`, `pytest`)
- Full error output (not truncated; include the first occurrence and the final summary)
- Environment details:
  - OS
  - runtime versions: Node/Python/JDK
  - package manager: npm/yarn/pnpm/pip/poetry/uv/gradle/maven
  - CI environment (container image, runner)
- Any recent changes:
  - dependency upgrades, lockfile edits, toolchain updates, config changes
- Whether this is a regression and last-known-good commit

## Context to attach (Cursor @ mentions)

Attach:

- @Terminal output for the failing command
- @Files for relevant config files:
  - JS/TS: package.json, tsconfig, lockfile
  - Python: pyproject.toml, requirements.txt, lockfile, tool config
  - Java: pom.xml, build.gradle, settings.gradle
  - Monorepos: workspace config, build scripts
- @Git diff for recent changes that might affect build
- @CI config if CI-only failure (pipeline YAML)

## Core principles (best practices)

1) Start from a clean, reproducible state (clean build).
2) Prefer fixing the underlying mismatch over pinning random versions.
3) Minimize blast radius: smallest change that makes build pass.
4) Keep dev and CI aligned: toolchain versions, lockfiles, and scripts.
5) Never commit secrets; never bypass security checks to “make build green”.

## Triage workflow (deterministic)

### Step 0: Classify the failure type

Pick one:
A) Syntax/compile error
B) Typecheck error
C) Dependency resolution / lockfile mismatch
D) Toolchain mismatch (wrong Node/Python/JDK)
E) Native module / platform-specific build error
F) Config error (tsconfig, babel, lint, gradle settings)
G) CI-only error (missing env, cache, permissions)

### Step 1: Re-run with maximum clarity

- Ensure you can reproduce with a single command.
- Re-run with verbose logging if supported (`--verbose`, `-X`, etc.).
- Capture:
  - first error location
  - the exact file/line
  - the tool that raised it

### Step 2: Reduce variables

- Use a clean working tree:
  - no uncommitted changes
  - reset local caches only if necessary
- Align versions:
  - confirm runtime versions
  - confirm package manager version
- If CI-only:
  - replicate CI image locally if possible
  - compare env vars and working directory structure

### Step 3: Identify root cause

- Locate the *first* meaningful error (often later errors are cascading).
- Determine whether it’s:
  - source code issue
  - config issue
  - dependency graph issue
  - environment mismatch

### Step 4: Apply minimal fix

- Make the smallest safe change that eliminates the root cause.
- Avoid “blanket” changes:
  - do not reformat the world
  - do not upgrade the entire dependency tree unless necessary

### Step 5: Verify and harden

- Run the smallest relevant checks first:
  - build
  - typecheck
  - lint (if part of build)
- Then run CI-equivalent checks if feasible.
- Add prevention:
  - pin toolchain versions
  - update docs/runbook
  - add CI guard if mismatch is common

## Output format (strict)

### 1) Build failure summary

- Failing command(s)
- Environment
- Primary error (first meaningful error)
- Secondary/cascading errors (if any)

### 2) Root cause

- Explain in plain terms
- Point to the exact file/config causing it
- If version mismatch, list the mismatched versions

### 3) Fix plan or patch

**If root cause is clear**

- List exact changes:
  - file paths
  - what to change
- Explain why it is the minimal fix

**If root cause is unclear**

- Provide an investigation plan:
  - specific files to inspect
  - commands to run
  - what evidence to collect

### 4) Verification commands

- Local verification (copy-paste)
- CI verification guidance
- Expected outputs (“build succeeds”, “no type errors”)

### 5) Guardrails (what NOT to do)

Tailor to the failure, examples:

- Do not disable typechecking
- Do not delete lockfile casually
- Do not add `--force` / `--legacy-peer-deps` unless you can justify and document
- Do not vendor binaries without review

### 6) Preventing recurrence

- Toolchain pinning strategy (examples):
  - `.nvmrc` / `.tool-versions` / runtime manager
  - lockfile enforcement in CI
  - “check versions” script
- Documentation updates

## Common patterns & clean fixes (tool-agnostic)

### Dependency mismatch

- Ensure lockfile matches package manager
- Fix peer dependency conflict by:
  - upgrading the dependent package responsibly
  - or using compatible versions (document why)
- Avoid: “just force install” unless temporary and tracked

### Toolchain mismatch

- Pin versions and enforce in CI
- Align local dev docs with CI image

### Generated code / build artifacts

- Ensure generation steps exist in CI and dev flow
- If committed, ensure they’re deterministic and reviewed

### Monorepo pitfalls

- Workspace hoisting differences
- Package boundary violations
- Unintended cross-package dependency

## Minimal “DoD”

- The build passes locally on the reported environment
- CI passes (or a clear reason why CI differs + plan to align)
- Root cause is explained and documented
- No risky shortcuts were introduced
- Any necessary docs updates are included

## Final instruction

End with:

- “I am ready to implement the minimal patch now” OR “I need the following evidence first”
- A prioritized list (max 6) of what the user should provide next
