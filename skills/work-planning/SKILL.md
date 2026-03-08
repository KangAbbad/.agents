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

1. **Research codebase** to understand domain and patterns
2. **Create directory**: `docs/feature-plan/{YYYYMMDD}/{specific-topic}/`
3. **Write README.md** with Purpose, Scope, Execution Order, Notes
4. **Create task files** (one per task): `task-01.md`, `task-02.md`, etc.
5. **Verify structure** matches examples in REFERENCE.md

### Creating a Refactor Plan

1. **Analyze code** to identify refactoring targets
2. **Create directory**: `docs/refactor-plan/{YYYYMMDD}/{specific-topic}/`
3. **Write README.md** with refactoring scope and risks
4. **Create task files** with **Refactoring Constraints** section
5. **Mark dependencies** in Execution Order

## Directory Naming Rules

**Must be descriptive and specific:**

| Good | Bad |
|------|-----|
| `table-factory-migration` | `refactor` |
| `customer-crud-endpoint-refactor` | `updates` |
| `payment-method-api-cleanup` | `fix-stuff` |

## Task File Essentials

Every task must include:

- **Workflow**: Load AGENTS.md, read business logic, execute steps, verify
- **Category**: route, component, hook, service, type, utility
- **Description**: Action-result, present tense
- **Files Affected**: Specific file paths
- **Steps**: Imperative verbs, checklist format
- **Passes**: `false` initially, `true` after verification

## Verification Checklist

Before marking task complete:

- [ ] Code changes made? → Run `bun typecheck`
- [ ] Files have unit tests? → Run `bun test:run`
- [ ] No violations? → Mark `**Passes:** true`
- [ ] Refactoring? → Preserved useEffect dependencies and EntityType

## References

- [Plan Structure](REFERENCE.md#plan-structure)
- [README Template](REFERENCE.md#readme-template)
- [Task File Template](REFERENCE.md#task-file-template)
- [Refactoring Constraints](REFERENCE.md#refactoring-constraints)
