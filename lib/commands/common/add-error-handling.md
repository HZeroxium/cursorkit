# /add-error-handling — Standardize error model, retries/timeouts, and user-facing messages

## Intent

You will improve robustness and developer experience by:

- Standardizing error contracts (HTTP/gRPC/internal)
- Adding safe retries/timeouts (where appropriate)
- Ensuring user-facing messages are clear and non-leaky
- Preserving security: do not expose sensitive internals

## When to use

- Errors are inconsistent across endpoints/modules.
- Callers can’t reliably handle errors (no stable codes).
- There’s retry chaos (random retries, duplicated side effects).
- Timeouts are missing, causing cascades and hung requests.
- Logs leak stack traces or secrets to clients.

## Required context (attach before running)

Attach:

- A few representative endpoints/RPC methods where errors occur.
- Current error responses (samples from logs or API clients).
- Any existing error middleware/filters/interceptors.
- Observability context: request IDs, trace IDs if available.
- Constraints: backward compatibility requirements.

If missing, ask for:

- 2–3 sample failing requests and current responses.

## Inputs (fill in)

- Protocol(s): [HTTP REST / GraphQL / gRPC / CLI]
- Client types: [Web / Mobile / Internal services]
- Backward-compat constraints: [STRICT/RELAXED]
- Retry policy constraints:
  - Are operations idempotent? [YES/NO/PARTIAL]
  - Rate limits? [YES/NO]
- Timeouts budget: [P95/P99], [GLOBAL DEADLINE]
- Error taxonomy preference: [DOMAIN CODES / HTTP STATUS ONLY / HYBRID]

## Output artifacts

1) Error model spec (1-page):
   - statuses / codes / messages / details
2) Implementation changes:
   - centralized error mapping middleware
   - typed/structured errors
3) Retry/timeout policy:
   - where to retry, where NOT to retry
4) Tests:
   - regression tests for error shape
5) Documentation:
   - examples of errors clients can expect

## Guardrails (do NOT do)

- Do NOT expose stack traces or SQL queries to clients.
- Do NOT retry non-idempotent operations by default.
- Do NOT swallow errors (no silent failures).
- Do NOT “catch all and return 200”.
- Do NOT invent new error formats per endpoint.

---

## Step 1 — Define a stable error taxonomy

Create a small taxonomy that works across the system:

- ValidationError (client fixable)
- AuthenticationError / AuthorizationError
- NotFound
- Conflict (optimistic lock / duplicate)
- RateLimited / QuotaExceeded
- UpstreamUnavailable / Timeout
- InternalError (unexpected)

For each category define:

- HTTP status (or gRPC status)
- Stable machine-readable `code`
- Human message (safe)
- Optional `retryable` flag
- Optional `correlationId` / `traceId`

### Recommended default for HTTP APIs: Problem Details (RFC 9457)

Define a standard response shape (example fields):

- type: URI identifying the problem type
- title: short summary
- status: HTTP status code
- detail: safe explanation
- instance: URI to identify this occurrence
- extensions: { code, correlationId, errors[], retryable, timestamp }

---

## Step 2 — Centralize error mapping

Implement a single error handler layer:

- REST: middleware/filter (framework-specific)
- GraphQL: formatError hook + extensions
- gRPC: server interceptor mapping to status codes + metadata

Rules:

- Domain errors -> mapped deterministically
- Unknown errors -> InternalError with generic message
- Always attach correlationId/traceId to response (if you have it)
- Always log the original error server-side

---

## Step 3 — Make retries safe (idempotency-first)

Classify operations:

- Safe to retry:
  - GET, HEAD
  - idempotent PUT/DELETE (if designed as such)
  - read-only RPCs
- NOT safe to retry by default:
  - POST create order/payment/charge
  - non-idempotent RPCs

Add idempotency mechanisms when needed:

- Idempotency-Key header
- request deduplication store
- exactly-once via outbox/inbox pattern (if available)

Retry policy recommendations:

- Use exponential backoff with jitter
- Cap max retries and total elapsed time
- Respect Retry-After header when provided
- Stop retrying on permanent errors (4xx validation/auth)

---

## Step 4 — Add timeouts (deadlines) and propagate them

Set timeouts at boundaries:

- client -> server request timeout
- server -> upstream call timeout
- DB timeout (statement/query timeout)

Propagate deadlines:

- HTTP: use consistent timeout budget and pass along (if your stack supports)
- gRPC: set deadlines and let them propagate; ensure cancellation is honored

Rules:

- No unlimited timeouts in production paths.
- Budget the request time across downstream calls (avoid “each hop gets full timeout”).

---

## Step 5 — User-facing messaging policy

Client messages must be:

- Actionable (“Try again later”, “Fix input fields”)
- Non-sensitive (no stack trace, no internal IDs except correlationId)
- Consistent across endpoints

Server logs may include:

- stack trace
- internal context
but must avoid secrets (tokens, passwords, full credit card, etc.)

---

## Step 6 — Tests that lock the contract

Add tests for:

- Error shape is stable (fields exist)
- Correct status/code mapping per category
- Retryable vs non-retryable behavior
- Timeout behavior for upstream calls (mocked)

At minimum:

- one unit test for the mapper
- one integration test for an endpoint demonstrating problem details

---

## Definition of Done

- [ ] A documented taxonomy exists
- [ ] A single centralized mapper is used
- [ ] Clients can rely on stable codes and shapes
- [ ] Timeouts exist and are propagated
- [ ] Retries are safe, limited, and justified
- [ ] Tests enforce the error contract

## Example invocation

"Standardize REST API errors using RFC 9457 Problem Details, add correlationId, define retry/timeout policy for upstream calls, and add tests to lock the contract."
