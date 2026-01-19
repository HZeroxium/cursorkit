# /codebase-tour

## Purpose

Give a high-signal architecture and navigation overview of the repository:

- Key entrypoints and how the app runs
- Module boundaries and ownership
- Data flow (request → processing → persistence → response)
- Configuration and environments
- Testing strategy and how to run the smallest checks
- “Where to make changes” map for common tasks

## When to use

Use when:

- You just opened a new repo or a new subsystem
- You’re onboarding someone (or yourself) quickly
- You need to locate the best place to implement a change
- You want to avoid wrong-file edits and spaghetti changes

## Ground rules

1) Prefer concrete file paths, not vague descriptions.
2) If uncertain, cite evidence: mention the file/function/class you saw.
3) Do not invent architecture. If you can’t find it, say so and propose how to discover it (search terms, commands).
4) Avoid massive dumps. High-signal summary + links to key files is the goal.

## Context gathering (do first)

1) Identify how to run/build/test:
   - Look for `README`, `package.json`, `pyproject`, `pom.xml`, `build.gradle`, `Makefile`, CI configs.
2) Identify entrypoints:
   - Server: routes/controllers, main bootstrap, dependency injection config
   - CLI: main command, subcommands
   - Frontend: app entry, routing, state management
3) Identify core domains:
   - Key data models/entities
   - Business logic services
   - Integration boundaries (DB, cache, queue, external APIs)
4) Identify cross-cutting concerns:
   - Auth, logging, metrics/tracing, error model
   - Validation and schema definitions
5) Identify test layout and conventions.

## Output format (strict)

### 1) TL;DR (10 bullets max)

- What this repo is
- How to run it
- Where the “center of gravity” is (main modules)
- Biggest risks / sharp edges

### 2) How to run / build / test (copyable commands)

Provide the smallest set of commands:

- Install/setup
- Run dev
- Run unit tests
- Run typecheck/lint/build
If you can’t confirm exact commands, provide the best guess and label it “Needs verification”.

### 3) Architecture map

**A) High-level components**

- List main components/modules and their responsibilities

**B) Module boundaries**

- What depends on what (boundaries)
- Where business logic lives vs infra

**C) Data flow (happy path)**

- Describe main flows with explicit file references where possible

Optional: include a Mermaid diagram (only if it adds clarity), e.g.:

- Request flow
- Component diagram
- Sequence diagram for a key action

### 4) Key directories and “what lives here”

For each top-level folder:

- Purpose
- What to change there
- What NOT to change there

### 5) Key files index (curated)

Provide a short list (10–25 items max):

- Entrypoints
- Configuration
- Core domain logic
- Persistence layer
- Integrations
- Tests

Format:

- `path/to/file` — why it matters

### 6) Conventions and standards

- Naming patterns
- Error handling conventions
- Logging/observability conventions
- Testing conventions
If you don’t see explicit conventions, say so and propose adding lightweight docs/rules.

### 7) “Where to implement X” quick guide

Give a mapping for common tasks, e.g.:

- Add a new API endpoint
- Add a DB table / migration
- Add a background job
- Add a UI screen / route
- Add a feature flag
- Add metrics/logging

### 8) Open questions / missing context

- What you couldn’t find quickly
- What you need from the user to be fully confident

## Quality bar

Before you finish:

- Ensure every major claim references at least one file path
- Avoid hand-wavy architecture statements
- Provide a clear next step (“If you want to change Y, start at file X”)
