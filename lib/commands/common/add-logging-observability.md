# /add-logging-observability — Add logs/metrics/tracing with best practices (no secrets)

## Intent

You will introduce or improve observability in a safe, consistent way:

- Structured logging with correlation IDs (trace_id/span_id if available)
- Metrics that are queryable and low-cardinality
- Tracing instrumentation using OpenTelemetry conventions
- Guardrails: never leak secrets or sensitive user data

## When to use

- Production incidents are hard to debug.
- Logs are unstructured or inconsistent.
- Metrics are missing / too noisy / too high cardinality.
- Traces are missing for critical flows.
- You need to connect "what happened" across services.

## Required context (attach before running)

Attach:

- Critical flows (endpoints/jobs/consumers) to instrument.
- Current logging setup (logger config, formatters).
- Metrics stack (Prometheus, OTEL Collector, vendor APM).
- Tracing setup (OTEL SDK present? existing propagators?).
- Security constraints (PII/PCI/PHI rules).

If missing, ask:

- Where logs go? (stdout, file, cloud logging)
- Where metrics go? (Prometheus scrape, OTEL export)
- Where traces go? (OTEL collector, vendor endpoint)

## Inputs (fill in)

- Observability goals: [DEBUGGING / SLOs / SECURITY AUDIT / COST CONTROL]
- Signals to add: [LOGS / METRICS / TRACES]
- Environments: [DEV / STAGING / PROD]
- PII policy: [STRICT/MODERATE]
- Sampling strategy: [HEAD / TAIL / RATE]

## Output artifacts

1) Instrumentation plan:
   - what to log, what metrics, what spans
2) Implementation changes:
   - middleware/interceptors
   - metric registry setup
   - trace provider setup
3) Naming conventions and field schema
4) Validation steps:
   - example queries and expected outputs
5) Security checklist (no secrets)

## Guardrails (do NOT do)

- Do NOT log secrets: tokens, passwords, session IDs, auth headers.
- Do NOT add high-cardinality labels (userId, orderId) to metrics.
- Do NOT create a metric per dynamic value.
- Do NOT trace everything without sampling or cost awareness.
- Do NOT rely on logs as a database (no dumping huge payloads).

---

## Step 1 — Define a structured logging schema (minimal but consistent)

Recommended baseline fields:

- timestamp
- level
- service/module
- message
- correlation_id (request_id)
- trace_id, span_id (if tracing enabled)
- http.method, http.route, http.status_code (if applicable)
- duration_ms
- error.type, error.code (if error)

Rules:

- Prefer JSON logs for machines; keep messages short but informative.
- Use consistent keys across services.
- Redact/sanitize sensitive fields (PII, secrets).

Add "log injection" defense:

- sanitize user-controlled strings where needed
- avoid logging raw headers/body by default

---

## Step 2 — Add correlation end-to-end

If tracing exists:

- inject trace_id/span_id into logs automatically via context
If tracing does NOT exist yet:
- generate request_id in inbound middleware and propagate it downstream

Propagation:

- HTTP: W3C Trace Context if using OTel propagators
- gRPC: use interceptors to propagate metadata/context

---

## Step 3 — Metrics: pick a small set of high-value metrics

Start with:

- Request rate (RPS)
- Error rate (5xx, domain failures)
- Latency (histogram)
- Saturation (queue depth, thread pool, DB connections)
- Business metrics (orders created, payments succeeded) — carefully scoped

Naming conventions:

- Use consistent unit suffixes (_seconds,_bytes)
- Prefer histograms for latency distributions
- Labels:
  - keep them bounded (method, route, status, service)
  - avoid user/session/order IDs

---

## Step 4 — Tracing: instrument critical path

Create spans around:

- inbound request handling
- DB calls (ORM usually has auto-instrumentation)
- outbound HTTP/gRPC calls
- message publish/consume
- long-running jobs stages

Attributes:

- follow semantic conventions when possible
- add domain attributes sparingly (bounded values only)

Sampling:

- Start with a conservative sampling rate in prod.
- Increase sampling temporarily during incidents.

---

## Step 5 — Where to implement (common patterns)

- Web servers: middleware / filters / interceptors
- Background jobs: wrap job execution with a span + duration metric
- SDK clients: client wrappers that record:
  - duration
  - retry counts
  - error codes
- Database layer: capture query timings; avoid logging full SQL with params in prod

---

## Step 6 — Validate with "golden queries"

Provide example checks:
Logs:

- filter by correlation_id
- filter by error.code
Metrics:
- top routes by p95 latency
- error rate by service
Traces:
- find trace by trace_id from a log entry
- confirm spans cover critical downstream calls

---

## Definition of Done

- [ ] Structured logs are emitted with stable fields
- [ ] Correlation works (request_id, trace_id if enabled)
- [ ] Metrics are low-cardinality and useful for SLOs
- [ ] Tracing covers critical flows with sane sampling
- [ ] No secrets/PII leaks (verified)
- [ ] Quick "how to debug" doc exists

## Example invocation

"Add OTel tracing + JSON structured logs with trace_id correlation for checkout flow, and add Prometheus-style metrics (requests_total, request_duration_seconds histogram) with safe labels only."
