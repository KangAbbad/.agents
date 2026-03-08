# API Test Collection Reference

Quick reference and index for Bruno scoped test flows.

---

## When to Update Collections

**Update when you:**
- Create new route in `src/routes/*/`
- Create new route module
- Change endpoint URL/path
- Add/remove/modify request body
- Add/remove query/path parameters
- Change response structure
- Delete an endpoint
- Register route in `src/index.ts`

**Self-check:** Did you touch `src/routes/` or `src/index.ts`? → Update collection.

---

## Quick Start

### Running a Flow

```bash
# Navigate to the flow directory
cd bruno-collections/{resource-collection}/{flow-folder}

# Run with the flow's own environment
bru run . --env-file environments/Local.bru

# Run with delay between requests
bru run . --env-file environments/Local.bru --delay 500

# Run with JSON output
bru run . --env-file environments/Local.bru -o results.json
```

---

## Guidelines Index

Detailed guides organized by tool:

### Bruno (Recommended)

#### [bruno/01-flow-patterns.md](guidelines/bruno/01-flow-patterns.md)
Common patterns for test flows:
- Create + Verify
- Create + Update + Verify
- Create + Delete + Verify 404
- List + Filter Verification
- Multi-Step Lifecycle
- Dependent Resources
- Error/Edge Case Verification

#### [bruno/02-templates.md](guidelines/bruno/02-templates.md)
Ready-to-copy templates:
- `bruno.json`
- `environments/Local.bru`
- `1-login.bru` (required first step)
- Action steps (POST/PATCH/DELETE)
- Verification steps (GET, expect 404)
- `flow.md`
- Parent collection README.md
- Complete flow example

#### [bruno/03-environment-variables.md](guidelines/bruno/03-environment-variables.md)
- Always required variables
- Resource-specific variables
- How variables are set (manual vs auto-set)
- Naming conventions
- Parent collection template

#### [bruno/04-troubleshooting.md](guidelines/bruno/04-troubleshooting.md)
Common issues and solutions:
- Login fails with 401
- Variable not set between steps
- Steps run out of order
- "Collection not found" error
- Assert failures
- Token expires mid-flow
- Environment file not found
- Request body issues

### Hoppscotch (Alternative)

#### [hoppscotch/01-basics.md](guidelines/hoppscotch/01-basics.md)
- Collection structure
- JSON escaping rules
- Request structure
- Auth configuration
- Test scripts
- Validation commands
- Common mistakes

---

## Assertion Reference

Bruno supports these assertion operators in `assert` blocks:

| Operator | Example | Description |
|----------|---------|-------------|
| `eq` | `res.status: eq 200` | Equals |
| `neq` | `res.status: neq 401` | Not equals |
| `gt` | `res.body.data.length: gt 0` | Greater than |
| `gte` | `res.body.data.length: gte 1` | Greater than or equal |
| `lt` | `res.body.data.length: lt 100` | Less than |
| `lte` | `res.body.data.length: lte 50` | Less than or equal |
| `isDefined` | `res.body.data.id: isDefined` | Value exists |
| `isUndefined` | `res.body.error: isUndefined` | Value doesn't exist |
| `isNull` | `res.body.data.deleted_at: isNull` | Value is null |
| `contains` | `res.body.data.name: contains "Outlet"` | String contains |
| `isString` | `res.body.data.name: isString` | Type check: string |
| `isNumber` | `res.body.data.total: isNumber` | Type check: number |
| `isBoolean` | `res.body.data.isShared: isBoolean` | Type check: boolean |
| `isArray` | `res.body.data: isArray` | Type check: array |

### Advanced Assertions with Scripts

```bru
script:post-response {
  const json = res.getBody();
  const { expect } = require('chai');

  expect(res.getStatus()).to.equal(200);
  expect(json.data).to.have.property('id');
  expect(json.data.name).to.include('Test');

  // Save for next step
  if (json.data && json.data.id) {
    bru.setVar("resourceId", json.data.id);
  }
}
```

---

## Checklist: Creating a New Flow

Use this checklist when creating a new scoped flow:

- [ ] **Flow folder** created with `kebab-case` name directly in the collection root (e.g., `{resource-collection}/{flow-folder}/`)
- [ ] **`bruno.json`** created inside flow folder (makes it a runnable collection)
- [ ] **`environments/`** directory created inside the flow folder
- [ ] **`environments/Local.bru`** created (gitignored) with real credentials (copy from parent `Local.bru.example`)
- [ ] **`1-login.bru`** copied from template (identical across all flows)
- [ ] **Action steps** numbered sequentially (`2-`, `3-`, `4-`, etc.)
- [ ] **Each action step** includes:
  - [ ] `meta` block with descriptive name and correct `seq`
  - [ ] Correct HTTP method and URL
  - [ ] `auth:bearer` block with `{{accessToken}}`
  - [ ] `assert` block with at minimum `res.status` check
  - [ ] `script:post-response` if it produces an ID needed by later steps
- [ ] **Verification step** asserts specific field values, not just status code
- [ ] **`flow.md`** documents purpose, steps, prerequisites, and expected outcome
- [ ] **Test run** — executed successfully with `bru run . --env-file environments/Local.bru` from the flow directory

---

## Key Design Principles

| Principle | Rationale |
|-----------|-----------|
| Flows are the **primary** testing method | Self-contained flows with auto-auth are easier to run than standalone files |
| Flows live **directly** in the collection root | Easy to find, no nested `flows/` directory |
| Numbered prefixes (`1-`, `2-`, `3-`) | Bruno runs files alphabetically — numbers enforce execution order |
| Login is **duplicated** per flow | Each flow is fully self-contained, no cross-dependencies |
| **Per-flow environments** | Each flow has its own `environments/` for complete isolation |
| `flow.md` per flow folder | Documents the purpose, steps, and expected outcomes |

---

## Full Documentation

For complete documentation, see:
- `docs/knowledge-base/09-bruno-scoped-flows.md` (project knowledge base)
- `SKILL.md` (this skill's main documentation)
- Guidelines in `guidelines/` directory
