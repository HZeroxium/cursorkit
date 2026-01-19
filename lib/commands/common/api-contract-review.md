# /api-contract-review

You are reviewing an API contract (OpenAPI/Swagger or equivalent).
Your goal: ensure the contract is consistent, evolvable, and backward-compatible, with a clean error model and pagination strategy.

## Required context (attach)

1) The API contract file(s): openapi.yaml/json, proto files, or endpoint docs.
2) Current deployed version + any existing clients (web/mobile/3rd party).
3) Change intent: new feature? bug fix? deprecation? performance?
4) Existing error format conventions and auth model (JWT/OAuth/session).
5) Any API governance rules (naming, versioning, pagination).

If missing, list what is missing and why it blocks the review.

## Output

A) Compatibility assessment (Breaking / Non-breaking / Ambiguous)
B) Issues list (ordered by severity): Security > Breaking > Data correctness > DX > Style
C) Concrete recommendations with examples
D) A short migration plan if breaking changes are unavoidable
E) Checklist for tests + monitoring after release

## Step-by-step review checklist (be explicit)

1) Surface area & ownership
   - List resources/endpoints touched.
   - Identify which are public vs internal.

2) Backward compatibility
   - Detect breaking changes: removed fields/endpoints, changed types, changed semantics, stricter validation, enum narrowing.
   - Prefer additive changes: new optional fields, new endpoints, new versions.
   - If deprecating, provide explicit deprecation policy: timeline, headers, docs.

3) Versioning strategy
   - Verify version placement: path (/v1), header, or media type.
   - Ensure versioning rules are consistent across endpoints.
   - Avoid “silent breaking changes” within the same version.

4) Pagination
   - Choose one: cursor-based (preferred for large/real-time datasets) or offset-based (simple, but can be inconsistent under writes).
   - Define:
     - request params: limit/page_size, cursor/after/before, sort order
     - response shape: items + next_cursor (and optionally prev_cursor)
     - max page size + default page size
   - Document stability guarantees: ordering must be deterministic.

5) Error model
   - Ensure a consistent, machine-readable error payload:
     - stable error code, human message, details, trace_id/request_id
   - Define mapping: validation errors, auth errors, not found, conflict, rate limit.
   - Do not leak secrets or internals.

6) Idempotency & retries
   - Identify which operations must be idempotent (PUT/DELETE, payment-like POST).
   - If supporting retries, define idempotency key header and semantics.

7) Data contracts & validation
   - Validate required vs optional fields.
   - Use clear formats: date-time, UUID, email, etc.
   - Avoid inconsistent nullability: choose either field omitted or null; document it.

8) Security & privacy
   - Authn/Authz: scopes/roles per endpoint.
   - PII handling: mask, minimize, and document.
   - Rate limiting: document response behavior.

9) DX & consistency
   - Naming: consistent resource names, consistent casing.
   - Examples: include request/response examples for major endpoints.
   - Status codes: consistent usage (200/201/204/400/401/403/404/409/422/429/5xx).

## What NOT to do

- Do not introduce breaking changes without version bump + migration plan.
- Do not change semantics “quietly” (same field name, different meaning).
- Do not ship pagination without deterministic ordering.
- Do not expose raw stack traces or internal error messages.

## Definition of Done

- Each issue has: severity, impacted clients, recommended fix.
- The contract includes: stable error model + pagination policy + versioning clarity.
- If any breaking change: documented migration plan and deprecation timeline.
