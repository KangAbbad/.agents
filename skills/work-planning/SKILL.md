---
name: work-planning
description: Create structured work plans with tasks for features or refactoring. Use when planning new features, organizing refactoring work, or breaking down complex changes into manageable tasks.
---

# Work Planning Skill

## Quick Start

Create a plan directory with README and task files:

```
docs/{feature-plan|refactor-plan}/{YYYYMMDD}/{descriptive-topic}/
├── README.md
├── task-01.md
├── task-02.md
└── ...
```

## Workflows

### Creating a Feature Plan

1. **Research codebase** to understand domain, stack, and patterns
2. **Classify project type** as frontend, backend, full-stack, or unknown
3. **Build the Decision Ledger** from the full discussion
4. **Create directory**: `docs/feature-plan/{YYYYMMDD}/{specific-topic}/`
5. **Write README.md** with Purpose, Scope, Execution Order, Notes, and Decision Ledger
6. **Create task files** (one per task): `task-01.md`, `task-02.md`, etc. with project-type verification only
7. **Run the Completeness Audit**; do not implement until it reports `COMPLETE`
8. **Verify structure** matches examples in REFERENCE.md

### Creating a Refactor Plan

1. **Analyze code** to identify refactoring targets, stack, and project type
2. **Classify project type** as frontend, backend, full-stack, or unknown
3. **Build the Decision Ledger** from the full discussion
4. **Create directory**: `docs/refactor-plan/{YYYYMMDD}/{specific-topic}/`
5. **Write README.md** with refactoring scope, risks, and Decision Ledger
6. **Create task files** with **Refactoring Constraints** section and project-type verification only
7. **Mark dependencies** in Execution Order
8. **Run the Completeness Audit**; do not implement until it reports `COMPLETE`

## Mandatory Decision Ledger

Before writing tasks, extract every decision from the complete discussion. Do not summarize away qualifiers, exceptions, rejected alternatives, or scope exclusions.

Record one row per atomic decision:

| ID | Status | Decision | Rejected alternatives / exclusions | Source | Exact destination | Task |
|----|--------|----------|------------------------------------|--------|-------------------|------|
| D-01 | confirmed / assumption / discovery | Exact rule and edge conditions | What is explicitly not included and why | Discussion or code evidence | `README.md#section` or `task-NN.md#section` | `task-NN.md` |

Rules:

- `confirmed`: explicitly agreed; preserve wording and nuance.
- `assumption`: required interpretation not confirmed; label it and obtain confirmation before `COMPLETE`.
- `discovery`: fact found in code/docs; cite exact path and line when known.
- Map every decision to an exact README/task section and assign at least one implementing or verifying task.
- Use exact known file paths. If unknown, add an explicit discovery step naming where and how to locate the path; never guess.
- Check explicitly for permissions matrix, lifecycle/state transitions, date boundaries and report basis, money/cash invariants, soft-delete/reversal behavior, uniqueness/concurrency, privacy, offline dependencies, reporting semantics, and UI language/copy. Record each as a decision, exclusion, assumption, or confirmed non-applicability.
- Compare ledger rows against each other. Resolve conflicting scope, terminology, behavior, ordering, and acceptance criteria before writing `COMPLETE`; do not silently choose one.

## Mandatory Completeness Audit

Run after README and all tasks are written. Audit from scratch in an independent pass, re-reading the original discussion rather than trusting the first extraction.

1. Create a decision matrix mapping every ledger ID to its exact README section, task, implementation step, verification step, and acceptance criterion.
2. Confirm every decision has a task assignment and testable acceptance criterion; no orphan decisions or ungrounded tasks.
3. Cross-check README, task files, execution order, dependencies, terminology, scope, exclusions, and acceptance criteria for contradictions.
4. Re-check all nuanced edge-case categories listed above and every rejected alternative.
5. Verify every task starts with `**Passes:** false`.
6. Report `INCOMPLETE` with missing/conflicting IDs and revise until no issue remains. Report `COMPLETE` only when every matrix cell is satisfied and contradictions are resolved.

**Gate:** No implementation may start until the independent audit reports `COMPLETE`. Plan creation is not complete before this gate.

## Directory Naming Rules

**Must be descriptive and specific:**

| Good | Bad |
|------|-----|
| `table-factory-migration` | `refactor` |
| `customer-crud-endpoint-refactor` | `updates` |
| `payment-method-api-cleanup` | `fix-stuff` |

## Task File Essentials

Every task must include:

- **Workflow**: Load AGENTS.md, inspect project type, read business logic, execute steps, verify
- **Project Type**: frontend, backend, full-stack, or unknown
- **Category**: route, component, hook, service, type, utility
- **Description**: Action-result, present tense
- **Files Affected**: Exact known paths, or explicit discovery steps when unknown
- **Decision IDs**: Every implemented or verified ledger decision assigned to the task
- **Acceptance Criteria**: Testable criteria for every assigned decision
- **Steps**: Imperative verbs, checklist format; include implementation and verification for each decision
- **Passes**: Always `false` in every newly created plan; implementation changes it only after verification

## Verification Checklist

Before creating task verification instructions:

- [ ] Inspect project stack/files
- [ ] Classify plan as frontend, backend, full-stack, or unknown
- [ ] Frontend? → Include E2E/browser tests for UI and API integration flows
- [ ] Backend? → Include API testing collections
- [ ] Full-stack? → Include both frontend and backend verification
- [ ] Unknown? → Ask user or use generic project verification
- [ ] Code changes made? → Run project typecheck
- [ ] Project uses unit tests for affected code? → Mention relevant unit tests as optional additional verification
- [ ] Avoid listing both frontend and backend guidance unless full-stack or unknown requires it
- [ ] Refactoring? → Preserve useEffect dependencies and EntityType
- [ ] Decision Ledger captures every confirmed decision, rejected alternative, exclusion, assumption, and discovery
- [ ] Every decision maps to an exact README/task section, task assignment, verification step, and acceptance criterion
- [ ] Unknown paths have explicit discovery steps; no guessed paths
- [ ] All task `Passes` values are `false`
- [ ] Independent completeness audit reports `COMPLETE`; otherwise implementation is blocked

## Completion Rule

A plan is complete only when the Decision Ledger has no unresolved assumptions or contradictions, the decision matrix has no empty cells, cross-file checks pass, all task `Passes` values remain `false`, and an independent audit reports `COMPLETE`. Never infer completeness from document presence or a summary review.

## References

- [Plan Structure](REFERENCE.md#plan-structure)
- [README Template](REFERENCE.md#readme-template)
- [Task File Template](REFERENCE.md#task-file-template)
- [Refactoring Constraints](REFERENCE.md#refactoring-constraints)
