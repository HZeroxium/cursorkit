# /extract-module — Extract a module/service/class along clear boundaries (keep API stable)

## Intent

You are about to extract functionality into a new module (or service) while keeping the external API stable and minimizing risk.  
Your job: produce a safe, incremental extraction plan + execute with minimal scope creep, preserving behavior.

## When to use

- A file/class/package grew too large (high coupling, low cohesion).
- A domain area needs independent deployment/ownership (module or microservice).
- You need to isolate a dependency (e.g., vendor SDK, database access) behind a boundary.
- You want to move toward a modular monolith or microservices gradually (no big rewrite).

## Preconditions (must satisfy before moving code)

1) You have a clear "boundary hypothesis" (what belongs together and why).
2) You can define an API contract at the boundary (public interfaces, DTOs, events).
3) You can run at least a minimal test suite and/or a smoke test.
4) You have a rollback strategy (git revert, feature flag, compatibility adapter).

## Required context (attach before running)

Attach:

- The current module(s) / folder(s) where the code lives.
- Any public API surfaces: HTTP routes, GraphQL schema, gRPC proto, public exports.
- Current dependency graph clues: imports, build files, DI wiring, package manifests.
- Test files for the extracted behavior (or a reproducible manual flow).
- Recent diffs touching this area (if available).

If any of these are missing, STOP and ask the user to attach them.

## Inputs (fill in)

- Extraction target: [MODULE_NAME]
- Boundary definition (what is in/out): [BOUNDARY_IN] / [BOUNDARY_OUT]
- Compatibility requirement:
  - Must keep existing API stable? (Yes/No)
  - Allowed deprecations? (None / Soft deprecate / Hard remove)
- Runtime constraints: [PERF/SLA], [SECURITY], [DEPLOYMENT]
- Languages/frameworks: [STACK]
- Timebox: [TIMEBOX]

## Output artifacts (what you must produce)

1) A step-by-step plan + file impact map.
2) A dependency/boundary analysis: what currently couples to what.
3) A migration approach (incremental):
   - Facade/Adapter
   - Anti-corruption layer
   - Strangler routing (if microservice)
4) The actual code changes (small diffs), plus updated tests/build config.
5) A short "How to validate" checklist.

## Safety guardrails (do NOT do)

- Do NOT change business behavior unless explicitly requested.
- Do NOT “rewrite everything” to match a new architecture.
- Do NOT move code without preserving an integration path (facade/adapter).
- Do NOT break public APIs silently (routes, exported functions, schemas).
- Do NOT introduce circular dependencies between new and old modules.
- Do NOT add new dependencies without justification.

---

## Step 0 — Clarify boundary + contract (ask if unclear)

Ask (and document answers):

- What is the module’s responsibility statement in one sentence?
- What are the explicit inputs/outputs of the boundary?
- Who are the callers? What do they need?
- What must remain backward-compatible?

If the user can’t answer, infer using code signals:

- Folder naming, domain terms, DB tables, endpoints, ownership conventions.
Then propose a boundary hypothesis and ask for confirmation.

---

## Step 1 — Map the current shape (Architecture snapshot)

Produce:

- Entrypoints: routes/controllers/handlers/jobs/consumers.
- Core domain types: entities/aggregates/value objects (or equivalent).
- Data access: repositories/DAOs/ORM models.
- Side effects: HTTP calls, messaging, filesystem, caches.
- Cross-cutting: logging, auth, validation.

Also list:

- Coupling points: common util imports, shared global state, “god services”, static singletons.
- Hidden contracts: implicit behaviors relied upon by callers.

Deliver a "File Impact Map" (example format):

- Will move:
  - path/a.ts -> new-module/src/a.ts
- Will create:
  - new-module/src/public-api.ts
  - new-module/src/adapters/old-system-adapter.ts
- Will update:
  - path/entrypoints/*.ts
  - build config / dependency injection wiring
- Will add tests:
  - new-module/test/*.spec.ts

---

## Step 2 — Choose the extraction pattern (default: incremental)

Pick one (explain why):

1) **Modular extraction (same deployable)**:
   - Create `modules/[MODULE_NAME]/`
   - Expose a stable public interface
   - Old code calls the new module through the interface
2) **Strangler-style extraction (separate deployable/service)**:
   - Create a new service boundary
   - Introduce routing/proxying gradually
   - Keep legacy API stable via gateway/facade
3) **Library isolation**:
   - Create a library package and keep its API stable
   - Version with semver rules, deprecate carefully

Default recommendation:

- Start as a module inside the same repo/runtime unless you must deploy independently.

---

## Step 3 — Design the boundary API (keep it boring)

Define:

- Public interfaces (functions/classes/services)
- DTOs / request-response types
- Error contracts (typed errors or problem details)
- Events emitted/consumed (if async)

Rules:

- No leaking internal ORM models across the boundary.
- No exposing low-level vendor SDK types; wrap them.
- Keep API small: 3–7 primary operations.
- Document invariants and performance expectations.

Deliver:

- `README.md` for the module with examples of usage.

---

## Step 4 — Implement extraction in small, reviewable diffs

Suggested commit slicing:

1) Introduce new module folder + empty public API scaffolding.
2) Add adapter layer in old code calling new module (no behavior change).
3) Move code incrementally:
   - move pure functions first
   - then domain logic
   - lastly side-effectful integrations
4) Update DI/wiring/build configs.
5) Cleanup: remove dead code paths, but ONLY after tests pass.

Tactics:

- Prefer “copy then switch” over “move then fix everything” if risk is high.
- Keep old entrypoints stable; change internals behind them.

---

## Step 5 — Verify: tests + behavior parity

Minimum validation:

- Run smallest relevant unit tests for extracted area.
- Run 1–3 integration/smoke flows that cover critical behavior.
- Verify backward compatibility:
  - request/response shape unchanged
  - status codes unchanged
  - error messages stable (or documented)

If tests are missing:

- Add a regression test first (even minimal).

---

## Step 6 — Documentation + rollout plan

Provide:

- "How to use new module" snippet.
- "What moved" and where.
- Rollback steps:
  - revert commits
  - feature flag off
  - route back to old handler

---

## Final deliverable checklist (Definition of Done)

- [ ] Boundary documented (responsibility + in/out)
- [ ] Public API defined and stable
- [ ] No circular deps; dependency direction is correct
- [ ] Tests pass; minimal smoke checks recorded
- [ ] Backward compatibility preserved (or explicitly approved changes)
- [ ] Rollback plan exists

## Example invocation

"Extract payment calculation logic into `modules/payments/` as `PaymentCalculator`, keep existing REST endpoints stable, and do it in 3 small commits with updated unit tests."
