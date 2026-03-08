# Work Planning Reference

## Plan Structure

### Directory Layout

```
docs/
├── feature-plan/
│   └── {YYYYMMDD}/
│       └── {descriptive-topic}/
│           ├── README.md
│           ├── task-01.md
│           ├── task-02.md
│           └── ...
└── refactor-plan/
    └── {YYYYMMDD}/
        └── {descriptive-topic}/
            ├── README.md
            ├── task-01.md
            └── ...
```

### Directory Naming

**Format:** `{descriptive-topic}`

**Rules:**
- Must convey purpose at a glance
- Use specific action + target pattern
- Research codebase first — never guess

| Quality | Example | Why |
|---------|---------|-----|
| ✓ Good | `table-factory-migration` | Specific action + target |
| ✓ Good | `customer-crud-endpoint-refactor` | Clear scope |
| ✗ Bad | `refactor` | Too vague |
| ✗ Bad | `updates` | Meaningless |
| ✗ Bad | `fix-stuff` | No specificity |

---

## README Template

Every plan **must** include a `README.md`:

```markdown
# {Plan Title}

## Purpose

{What this plan achieves and why — 1-2 sentences}

## Scope

- {Area 1 of codebase affected}
- {Area 2 of codebase affected}
- {Any files, modules, or components}

## Execution Order

1. `task-01.md` — {brief description of what this task does}
2. `task-02.md` — {brief description} (depends on task-01)
3. `task-03.md` — {brief description} (independent)

## Notes

- {Any relevant context, risks, or decisions worth capturing}
- {Known issues or constraints}
- {Dependencies on other work}
```

### Required Sections

| Section | Purpose |
|---------|---------|
| **Purpose** | What and why (1-2 sentences) |
| **Scope** | Which areas of codebase are affected |
| **Execution Order** | Recommended task sequence with dependencies |
| **Notes** | Context, risks, decisions |

---

## Task File Template

**Filename:** `task-01.md`, `task-02.md`, etc. (sequential numbering)

```markdown
**Workflow:**

- Load `AGENTS.md` for project-wide guidelines (architecture, patterns, constraints)
- Load `docs/code-standard/README.md` and only referenced docs needed for the current task
- Read existing business logic first
- Execute steps in order
- **If code changes made**: Run `bun typecheck` to verify
- **If changed files have unit tests**: Run `bun test:run` to verify tests pass
- **If no violations found**: Skip typecheck/tests, mark as passed
- Set `**Passes:** true` after verification (or when no violations exist)

**Refactoring Constraints:**

- Do NOT add/remove useEffect dependencies - keep as-is
- Do NOT rename/rephrase `type EntityType`

**Passes:** false

**Category:** {route|component|hook|service|type|utility}

**Description:** {Action-result, present tense, one line}

**Files Affected:**

- `{path/to/file-1}`
- `{path/to/file-2}`

**Steps:**

- {Imperative verb step 1}
- {Imperative verb step 2}
- {Imperative verb step 3}
```

### Task Fields Reference

| Field | Format | Required |
|-------|--------|----------|
| **Workflow** | Bullet list of execution instructions | ✓ Always |
| **Refactoring Constraints** | Critical rules to preserve | Only for refactor plans |
| **Passes** | `false`, `true`, or `skipped - {reason}` | ✓ Always |
| **Category** | `route`, `component`, `hook`, `service`, `type`, `utility` | ✓ Always |
| **Description** | Action-result, present tense, one line | ✓ Always |
| **Files Affected** | List of specific file paths | ✓ Always |
| **Steps** | Imperative verbs, checklist format | ✓ Always |

### Category Definitions

| Category | Description | Example |
|----------|-------------|---------|
| **route** | Remix/React Router route files | `app/routes/dashboard.tsx` |
| **component** | React components | `app/components/Button.tsx` |
| **hook** | Custom React hooks | `app/hooks/useAuth.ts` |
| **service** | API service functions | `app/services/customer.ts` |
| **type** | TypeScript type definitions | `app/types/customer.ts` |
| **utility** | Helper/util functions | `app/utils/format.ts` |

---

## Required Workflow

### Before Starting

1. **Open task file** before coding
2. **Load `AGENTS.md`** for project-wide guidelines
3. **Load referenced docs** from `docs/code-standard/` as needed
4. **Read existing business logic** first

### During Execution

5. **Execute steps in order**
6. **Verify changes**:
   - Code changes made? → Run `bun typecheck`
   - Changed files have unit tests? → Run `bun test:run`
   - No violations found? → Skip verification, mark as passed

### After Completion

7. **Set `Passes: true`** after verification (or when no violations exist)

---

## Refactoring Constraints

**CRITICAL: When building refactoring plans ONLY, preserve:**

### 1. useEffect Dependencies

**Do NOT add or remove dependencies. Keep as-is.**

```typescript
// ❌ WRONG - Adding dependency
useEffect(() => {
  fetchData()
}, [fetchData, newDependency]) // Don't add newDependency

// ❌ WRONG - Removing dependency
useEffect(() => {
  fetchData()
}, []) // Don't remove existing dependencies

// ✅ CORRECT - Keep as-is
useEffect(() => {
  fetchData()
}, [fetchData])
```

### 2. EntityType Type

**Do NOT rename or rephrase this type name.**

```typescript
// ❌ WRONG - Renaming
type CustomerType = { ... }

// ❌ WRONG - Rephrasing
type EntityDataType = { ... }

// ✅ CORRECT - Keep exact name
type EntityType = { ... }
```

### 3. React Keys in .map() Loops

**When replacing `key={index}`:**

**ONLY use guaranteed unique identifiers:**
- `id`
- `value`
- `key`

**NEVER use potentially duplicable fields:**
- `name`
- `address`
- `label`
- `title`

```typescript
// ❌ WRONG - Using index
items.map((item, index) => <div key={index}>...</div>)

// ❌ WRONG - Using potentially duplicate field
items.map((item) => <div key={item.name}>...</div>)

// ✅ CORRECT - Using guaranteed unique identifier
items.map((item) => <div key={item.id}>...</div>)
```

---

## Passes Field Values

| Value | Meaning | When to Use |
|-------|---------|-------------|
| `false` | Pending verification | Initial state, task not started |
| `true` | Verified complete | After typecheck/tests pass |
| `skipped - {reason}` | Intentionally skipped | No code changes needed, or task deprecated |

### Examples

```markdown
**Passes:** false
```

```markdown
**Passes:** true
```

```markdown
**Passes:** skipped - no code changes required, only documentation updated
```

---

## Complete Example

### Plan Directory

```
docs/refactor-plan/20240115/customer-table-factory/
├── README.md
├── task-01.md
├── task-02.md
└── task-03.md
```

### README.md

```markdown
# Customer Table Factory Migration

## Purpose

Migrate customer list page from inline table to useTableFactory hook for consistency with other list pages.

## Scope

- Customer index route
- Customer query hooks
- Customer service layer

## Execution Order

1. `task-01.md` — Analyze current table implementation
2. `task-02.md` — Create column definitions using factory pattern (depends on task-01)
3. `task-03.md` — Replace table and verify types (depends on task-02)

## Notes

- Keep existing styling and behavior
- Ensure pagination still works
- Test with empty state
```

### task-01.md

```markdown
**Workflow:**

- Load `AGENTS.md` for project-wide guidelines
- Load `docs/code-standard/README.md`
- Read existing customer route implementation
- Execute steps in order
- **If code changes made**: Run `bun typecheck` to verify
- Set `**Passes:** true` after verification

**Refactoring Constraints:**

- Do NOT add/remove useEffect dependencies - keep as-is
- Do NOT rename/rephrase `type EntityType`
- When replacing key={index}, only use id/value/key, never name/address/label/title

**Passes:** false

**Category:** route

**Description:** Analyze current customer table implementation

**Files Affected:**

- `app/routes/_dashboard.customers._index/route.tsx`

**Steps:**

- Read current route component
- Identify table columns and their definitions
- Document current pagination logic
- Note any custom cell renderers
- List all useEffect dependencies currently used
```

---

## Planning Checklist

### Before Creating Plan

- [ ] Researched codebase for domain terms and patterns
- [ ] Chosen descriptive, specific topic name
- [ ] Determined if feature or refactor
- [ ] Identified all affected areas

### Plan Structure

- [ ] Directory follows naming convention: `{YYYYMMDD}/{descriptive-topic}`
- [ ] README.md has all required sections
- [ ] Execution order includes dependencies
- [ ] Each task is in separate file

### Task Files

- [ ] All task files have required fields
- [ ] Category is valid
- [ ] Description uses action-result format
- [ ] Steps use imperative verbs
- [ ] Files Affected lists specific paths
- [ ] Passes starts as `false`
- [ ] Refactoring constraints included (if refactor plan)

### Completion

- [ ] All tasks executed in order
- [ ] Typecheck run for code changes
- [ ] Tests run for affected files
- [ ] All passes marked `true` or `skipped`
