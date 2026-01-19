# /performance-review — Identify performance hotspots, propose profiling plan, and deliver quick wins

## Intent

You will run a disciplined performance review:

- Reproduce the performance issue (or approximate with benchmarks)
- Identify hotspots using appropriate profilers
- Recommend high-impact fixes with minimal risk
- Provide validation steps and regression guardrails

## When to use

- Latency is high (p95/p99), CPU is high, memory spikes.
- A recent change introduced slowdown.
- Throughput is insufficient; queues back up.
- UI feels janky; runtime performance issues.
- Cost is too high for the current workload.

## Required context (attach before running)

Attach:

- The symptom: latency numbers, CPU/memory graphs, slow endpoints.
- Workload pattern: QPS, payload sizes, concurrency level.
- Environment: local/dev/prod differences, hardware constraints.
- Any logs/traces/metrics that show slow paths.
- A minimal reproducer if possible (curl command, script, test).

If missing, ask for:

- a single most painful endpoint/flow
- one representative input causing slowness

## Inputs (fill in)

- Target component: [SERVICE/MODULE/UI]
- Primary symptom: [LATENCY/CPU/MEM/IO/DB]
- SLA/SLO: [P95/P99 targets]
- Languages: [JS/TS/Python/Java/Go/etc]
- Risk tolerance: [LOW/MED/HIGH]
- Timebox: [TIMEBOX]

## Output artifacts

1) Profiling plan (tool + commands + what to look for)
2) Hotspot list (ranked by impact)
3) Fix proposals (quick wins first)
4) Validation plan + regression checks
5) Optional: perf tests/benchmarks

## Guardrails (do NOT do)

- Do NOT optimize blindly without measurements.
- Do NOT micro-optimize cold paths.
- Do NOT trade correctness for speed.
- Do NOT introduce complex caching without invalidation plan.
- Do NOT remove safety/security checks unless approved.

---

## Step 1 — Establish baseline and measurement strategy

Define:

- "Before" metrics: p50/p95/p99 latency, CPU%, memory RSS, GC time, DB query times.
- Where measured: local vs staging vs prod.
- Inputs: representative requests and dataset sizes.

Create a simple repeatable benchmark:

- 10–100 iterations of the critical flow
- record timings
- keep conditions stable

---

## Step 2 — Pick the right profiler (by stack)

Guidelines:

- Web/Frontend: Chrome DevTools Performance record -> analyze main thread, scripting, rendering, long tasks.
- Python: cProfile/profile + pstats -> find hottest functions by cumulative time.
- Linux native/system-level: perf record/report -> identify CPU hotspots, kernel vs user space.
- Java: JFR (Flight Recorder) + JMC -> find allocation hotspots, lock contention, GC pressure, slow IO.

If the stack is mixed:

- start at the boundary where the symptom is observed, then go downstream.

---

## Step 3 — Identify hotspot categories

Classify hotspots:

1) Algorithmic inefficiency:
   - O(n^2) loops, redundant computations
2) IO-bound:
   - slow DB queries, N+1 patterns, excessive remote calls
3) Contention:
   - locks, synchronized blocks, thread pool starvation
4) Memory/GC:
   - excessive allocations, large objects, cache blowups
5) Serialization/parsing:
   - JSON parsing, protobuf, repeated transformations
6) Frontend runtime:
   - long tasks, layout thrashing, heavy re-renders

---

## Step 4 — Quick wins (ordered by typical ROI)

Choose the safest high-impact improvements first:

- Reduce N+1 queries; add batching.
- Add indices or optimize query plans (validate with EXPLAIN).
- Cache at the right layer with clear invalidation (or short TTL).
- Avoid repeated parsing/serialization; reuse objects.
- Limit payload sizes; paginate; stream when large.
- Improve concurrency:
  - async IO where appropriate
  - bounded queues and backpressure
- Frontend:
  - memoize heavy computations
  - virtualize lists
  - reduce re-render scope

For each fix:

- Provide expected impact
- Provide risk assessment
- Provide rollback plan

---

## Step 5 — Validate and prevent regressions

Validation checklist:

- Benchmark "after" under same conditions
- Confirm correctness (golden outputs)
- Confirm resource usage (CPU/mem)
- Add perf regression tests if feasible:
  - microbench for critical function
  - load test for endpoint (lightweight)

If production exists:

- propose a safe rollout:
  - canary
  - feature flag
  - compare metrics

---

## Definition of Done

- [ ] Baseline measurements recorded
- [ ] Profiling data collected (or credible plan + partial evidence)
- [ ] Hotspots ranked by impact
- [ ] Fix proposals prioritized (quick wins first)
- [ ] Validation plan and rollback steps documented
- [ ] Optional regression checks added

## Example invocation

"Review API latency regression after commit X: profile Python handlers with cProfile, identify top 3 hotspots, propose minimal-risk fixes and validation steps."
