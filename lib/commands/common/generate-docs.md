# /generate-docs

You are generating documentation for code changes.
Goal: produce accurate, minimal-but-sufficient docs that reflect real behavior (no hallucinations).

# Required context (attach)

1) The code being documented (files + relevant interfaces/types).
2) Existing documentation conventions in this repo (README style, docstring style, JSDoc rules).
3) Public API surfaces: exported functions/classes, endpoints, CLI commands.
4) Any lint rules for docs (docstring linter, typedoc, jsdoc config, etc).

# Output options (choose what fits)

- Docstrings (Python/Java/Kotlin/etc) for public modules/classes/functions
- JSDoc/TSDoc comments for exported TS/JS APIs
- README snippets: setup, run, test, env vars
- Examples: usage snippets, sample requests/responses
- Inline comments ONLY when non-obvious (avoid narrating trivial code)

# Global rules

- Only document what you can prove from the code in context.
- If behavior is unclear, ask for missing context or mark "TBD" explicitly.
- Keep docs consistent with house style and tooling.

# Step-by-step workflow

1) Identify "public" surfaces
   - Anything exported/public, or used by other modules/services, or externally consumed.
   - Skip purely private helpers unless they are complex or easy to misuse.

2) Extract truth from code
   - Inputs, outputs, side effects, errors thrown, performance characteristics.
   - Edge cases and invariants.
   - Threading/async behavior if relevant.

3) Produce docstrings (Python guidance)
   - Use triple quotes and keep a one-line summary.
   - Document parameters, return values, raises, and important side effects.
   - If the repo follows Google style, use sections like Args/Returns/Raises.

4) Produce JSDoc
   - Use @param to describe name/type/meaning.
   - Use @returns for return value, including Promise shape if async.
   - Include @throws when exceptions are part of API contract.
   - Provide @example for non-trivial APIs.

5) Generate README snippets
   - "Quickstart" first: install, configure, run.
   - Then "How to test" and "Troubleshooting".
   - Document environment variables with defaults and examples.
   - Include minimal diagrams only if needed.

6) Add examples
   - Provide example calls using real types and realistic values.
   - Ensure examples are consistent with code paths.

# What NOT to do

- Do not document imaginary options/flags/endpoints.
- Do not add long prose where a short example is clearer.
- Do not duplicate documentation across multiple places without a single source of truth.

# Definition of Done

- Docs compile/render cleanly in existing doc tooling.
- Public APIs have doc coverage consistent with team standard.
- Examples match actual types and behavior in code.
