# /lint-format-fix

## Purpose

Bring code into compliance with linting/formatting conventions with minimal, reviewable diffs.
This command outputs:

- A scoped lint/format plan (or patch) that avoids “format-the-world”
- Fixes that align with existing rules, not random stylistic choices
- Guidance for running the correct formatter/linter commands
- Prevention steps (pre-commit hooks, CI checks) if needed

## When to use

Use when:

- Lint errors block PR/CI
- Formatting drift makes reviews painful
- You’ve changed code and now lint rules complain
- You want to standardize style in a small area

Avoid when:

- The repo has no agreed lint/format setup (first define conventions)
- You’re about to do a major refactor—keep style fixes separate
- You’re tempted to reformat the entire repo in the same PR (usually bad)

## Non-negotiable rules

1) Scope first: only format/lint what’s necessary (changed files or a targeted folder).
2) Prefer auto-fix tools *when configured by the repo*; do not invent new rules.
3) Do not disable rules casually (`eslint-disable`, `# noqa`) unless justified.
4) Avoid noisy diffs. Separate formatting-only commits from logic changes.
5) Always run the smallest relevant checks after changes.

## Inputs (ask if missing)

- Which linter/formatter is used? (e.g., ESLint/Prettier, Ruff/Black, Checkstyle/Spotless)
- What command does CI run? (copy from CI logs or package scripts)
- Which files are failing? (file list + errors)
- Are we allowed to auto-fix? (some repos restrict)
- Any style exceptions? (generated code, vendored files)

## Context to attach (Cursor @ mentions)

Attach:

- @Terminal output from lint/format step (CI or local)
- @Config files (as applicable):
  - `.eslintrc*`, `eslint.config.*`, `.prettierrc*`
  - `pyproject.toml`, `ruff.toml`, `setup.cfg`
  - `checkstyle.xml`, `spotless.gradle`, editorconfig
  - `.editorconfig`
- @Files: the failing files

## Output format (strict)

### 1) Lint/format status summary

- Tools involved
- Failing command(s)
- File list with top error categories

### 2) Scope decision

Choose scope explicitly:

- Option A (preferred): changed files only
- Option B: targeted directory/module
- Option C: full repo (only with explicit approval)
State why the chosen scope is appropriate.

### 3) Fix strategy (ranked)

Rank by lowest risk and smallest diff:

1) Run auto-fixers (repo-provided)
2) Manual fixes for non-auto-fixable issues
3) Config adjustments (only if rules are wrong/outdated)
4) Rule suppression (last resort, documented)

### 4) Concrete actions (copy-paste commands)

Provide commands the developer can run.
If unknown, provide best guess labeled “Needs verification”.
Examples (placeholders):

- `npm run lint -- --fix`
- `pnpm lint --fix`
- `ruff check . --fix`
- `black .`
- `mvn spotless:apply`
- `./gradlew spotlessApply`

### 5) Manual fix guidance (if needed)

For each common error class, include:

- What it means
- The best fix pattern
- What NOT to do

Common categories:

- Unused imports / variables
- Type issues / explicit any
- Complexity / nested conditionals
- Naming conventions
- Formatting: line length, quotes, trailing commas
- Equality / null checks
- Async pitfalls (unhandled promises)
- Security lint warnings (unsafe eval, unsanitized input)

### 6) Config change rules (if config must change)

Only modify lint/format config when:

- CI/tool versions changed
- The current rule conflicts with repo standards
- The rule causes false positives and is documented
If you propose config changes:
- Explain why
- Show minimal diff
- Mention compatibility with CI and editor integrations

### 7) Verification checklist

- Run formatter/linter
- Run typecheck if lint depends on types
- Run smallest unit tests (if any logic touched)
- Confirm no unrelated files changed (diff review)

### 8) Prevention (optional)

Suggest one or two:

- Pre-commit hooks (lint-staged, pre-commit, etc.)
- Editor auto-format on save
- CI guard: fail if formatting differs
- “Format changed files only” scripts

## Anti-patterns (explicit)

- Reformat entire repo inside a feature PR
- Add disable comments to silence real issues
- Change lint rules to accommodate bad code
- Ignore security lint warnings without review
- Auto-fix generated code and commit it unintentionally

## DoD

- Lint/format passes locally using the same command as CI
- Diff is scoped and reviewable
- No new rule suppressions without justification
- If config changed, CI/editor behavior is still consistent

## Final instruction

End with:

- The smallest command sequence the user should run
- The expected results
- Any remaining questions about tooling/version alignment
