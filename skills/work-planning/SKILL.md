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
3. **Create directory**: `docs/feature-plan/{YYYYMMDD}/{specific-topic}/`
4. **Write README.md** with Purpose, Scope, Execution Order, Notes
5. **Create task files** (one per task): `task-01.md`, `task-02.md`, etc. with project-type verification only
6. **Verify structure** matches examples in REFERENCE.md

### Creating a Refactor Plan

1. **Analyze code** to identify refactoring targets, stack, and project type
2. **Classify project type** as frontend, backend, full-stack, or unknown
3. **Create directory**: `docs/refactor-plan/{YYYYMMDD}/{specific-topic}/`
4. **Write README.md** with refactoring scope and risks
5. **Create task files** with **Refactoring Constraints** section and project-type verification only
6. **Mark dependencies** in Execution Order

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
- **Files Affected**: Specific file paths
- **Steps**: Imperative verbs, checklist format
- **Passes**: `false` initially, `true` after verification

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

## References

- [Plan Structure](REFERENCE.md#plan-structure)
- [README Template](REFERENCE.md#readme-template)
- [Task File Template](REFERENCE.md#task-file-template)
- [Refactoring Constraints](REFERENCE.md#refactoring-constraints)
