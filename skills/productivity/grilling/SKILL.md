---
name: grilling
description: Grill the user relentlessly about a plan or design. Use when the user wants to stress-test a plan before building, or uses any 'grill' trigger phrases.
---

# Grilling

Interview the user relentlessly until shared understanding is complete. Traverse design branches and resolve dependencies one decision at a time. For every question, provide a recommended answer and ask exactly one question; wait for its answer before continuing.

## Decision Ledger

Maintain one persistent Decision Ledger throughout grilling. Record each confirmed decision immediately; never rely on conversational history. Each row contains:

| ID | Topic | Exact confirmed rule | Rationale | Rejected alternatives | Scope exclusions | Unresolved dependencies | Status | Source discussion context |
|----|-------|----------------------|-----------|----------------------|------------------|-------------------------|--------|---------------------------|
| D-01 | ... | Preserve exact wording and edge conditions | If stated | Explicitly rejected options | Explicit non-goals | Decision IDs or questions | confirmed / assumption / discovery | Question and answer, with turn/context |

Use `confirmed` only for explicit user agreement, `assumption` for interpretations awaiting confirmation, and `discovery` for facts established from codebase/docs. Investigate codebase facts instead of asking; record path and line when known. Keep the ledger authoritative and update existing rows when decisions change.

Whenever a new answer may alter an earlier decision, run a contradiction check before asking anything else. Identify affected IDs, present the conflict and recommended resolution in the next single question, then update the authoritative row; do not duplicate or silently overwrite decisions. Keep unresolved decisions as one-at-a-time questions.

## Coverage Gate

Before declaring discussion complete, audit the ledger and discussion against every category: domain model; relationships; lifecycle/state transitions; permissions; dates/timing; money/accounting invariants; validation; uniqueness/concurrency; deletion/reversal; privacy/security; offline behavior; reports; affected existing features; UI language/copy; testing; explicit non-goals. Record each category as a decision, discovery, assumption, or confirmed non-applicability. Ask remaining unresolved items one at a time. Completion requires no unreviewed category, no unresolved contradiction, and no unacknowledged assumption.

Do not create a work plan during grilling unless the user explicitly asks. When handing off to `work-planning`, export the complete ledger into the feature plan's `Decision Ledger`; preserve IDs, exact rules, rationale, rejected alternatives, exclusions, dependencies, status, and source context. Require every ledger row to trace to an exact plan section, task, verification step, and acceptance criterion, as required by `work-planning`.

Do not enact the plan until the user confirms shared understanding.
