# /write-adr

You are writing an Architecture Decision Record (ADR).
Goal: capture an architecturally significant decision with context, alternatives, and consequences.

## Required context (attach)

1) Decision topic (one sentence): e.g., "Adopt queue-based async jobs for video rendering".
2) Drivers:
   - Functional requirements
   - Non-functional requirements (SLOs, latency, cost, security, compliance)
3) Constraints: timeline, team skill, platform limits.
4) Alternatives considered (at least 2) and why they were not chosen.
5) Consequences: operational burden, migration cost, future flexibility.

If missing, ask for only the minimum needed to proceed.

## Output

- A single ADR file in Markdown, ready to commit.

## File naming convention (suggested)

- docs/adr/NNNN-short-title.md
- NNNN is sequential (0001, 0002, ...)

## ADR template (fill it precisely)

Title: NNNN: <Decision title>
Status: Proposed | Accepted | Deprecated | Superseded
Date: YYYY-MM-DD

## Context

- What problem are we solving?
- What forces/constraints matter (scalability, latency, security, maintainability)?
- What is the current state and pain?

## Decision

- The decision we made, stated clearly.
- Scope boundaries: what this affects and what it does NOT affect.
- Key design points and invariants.

## Alternatives Considered

For each alternative:

- Summary
- Pros
- Cons / risks
- Why not chosen

## Consequences

- Positive outcomes
- Negative trade-offs
- Operational impacts (on-call, monitoring, runbooks)
- Migration / rollout implications
- Follow-ups / action items

## Appendix (optional)

- Links to relevant docs, PRs, diagrams, benchmarks.

## Quality bar

- Must be understandable by a new team member in 5 minutes.
- Must include at least one rejected alternative with real reasoning.
- Must include operational consequences (not only code structure).

## What NOT to do

- Do not write an ADR for trivial refactors.
- Do not omit consequences ("no downside")—be honest about trade-offs.
- Do not copy generic text; make it specific to this system.

## Definition of Done

- ADR answers: Why now? Why this? Why not the others?
- ADR includes next steps and ownership if actions are required.
