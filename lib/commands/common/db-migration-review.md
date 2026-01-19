# /db-migration-review

You are reviewing database migrations for safe production rollout.
Goal: minimize downtime, locks, and rollback risk while preserving backward compatibility.

# Required context (attach)

1) Migration files (SQL/Flyway/Liquibase/etc) + ordering.
2) DB engine/version (Postgres/MySQL/etc) and traffic profile (QPS, table sizes).
3) Application release plan: single deploy vs phased deploy, blue/green/canary.
4) Current schema + how app reads/writes involved tables (queries, ORM models).
5) Any SLO constraints (max lock time, max replication lag, maintenance windows).

# Output

A) Risk assessment (Locking risk, Data risk, Performance risk, Rollback risk)
B) Review comments per migration (line-level if possible)
C) Safe rollout plan (Expand/Contract or equivalent)
D) Backfill plan (batching, idempotency, resume strategy)
E) Verification plan + rollback plan

# Principles (enforce)

- Prefer additive, backward-compatible schema changes first.
- Separate schema change from data backfill when possible.
- Avoid long blocking locks on hot tables.
- Always consider online index creation strategy if supported.

# Step-by-step workflow

1) Build a table-by-table impact map
   - Identify hot tables (writes during business hours).
   - Identify largest tables (risk of long operations).
   - Identify queries/paths that will break if schema changes are not backward-compatible.

2) Validate ordering & backward compatibility
   - Expand step: add nullable columns, new tables, new indexes (non-blocking strategy), triggers if needed.
   - App deploy step: write to both old and new columns (dual write) if migrating.
   - Backfill step: fill historical data in batches with checkpoints.
   - Contract step: remove old columns/constraints only after safe cutover.

3) Index strategy review (Postgres example)
   - If creating indexes on large/hot tables:
     - By default, index creation allows reads but blocks writes until finished.
     - Prefer concurrent index builds where available; understand caveats.
   - Ensure index naming conventions and purpose are documented.
   - Verify indexes match query patterns (leading columns, selectivity).

4) Constraint strategy
   - Adding NOT NULL / FK can block or scan large tables; consider phased approach:
     - Add constraint NOT VALID (if supported), validate later.
     - Add NOT NULL after backfill.
   - Ensure constraint introduction is compatible with current data.

5) Data backfill plan (must be explicit)
   - Batch size, rate limiting, and impact on production.
   - Idempotent updates and resume points.
   - Observability: progress metrics, error logs, dead-letter strategy.

6) Rollback plan
   - Define rollback per step:
     - Schema-only rollbacks might be hard; ensure forward-fix strategy exists.
     - Avoid irreversible destructive changes until confident.

7) Verification
   - Pre-deploy: run migrations on staging with production-like data scale if possible.
   - Post-deploy: smoke queries, replication lag checks, error rate monitoring.
   - Validate index usage (EXPLAIN) after rollout.

# What NOT to do

- Do not add heavy indexes on hot tables without an online strategy.
- Do not combine schema change + huge backfill in one migration transaction if it can lock tables.
- Do not drop columns/tables in the same release as introducing replacements.
- Do not assume "works on dev" implies safe on production scale.

# Definition of Done

- Each migration has: risk notes, expected runtime, lock behavior expectations.
- A phased rollout plan exists (expand -> deploy -> backfill -> contract).
- Rollback or forward-fix strategy documented.
- Verification checklist ready for on-call/runbook.
