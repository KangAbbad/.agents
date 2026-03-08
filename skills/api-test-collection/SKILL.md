---
name: api-test-collection
description: Create and maintain Bruno scoped test flows or Hoppscotch API test collections. Use when adding new API endpoints, creating route modules, or updating existing collections.
---

# API Test Collection Skill

## Quick Start

Choose your tool:

**Bruno** (CLI, file-based) — **RECOMMENDED**:
- Location: `bruno-collections/{resource-collection}/`
- Format: `.bru` files in numbered flow folders
- Best for: Version control, CI/CD, automated testing
- **Key concept**: Scoped flows that chain requests with auto-authentication

**Hoppscotch** (GUI, JSON):
- Location: `hoppscotch-collections/{entity-name}-collection.json`
- Format: Single JSON file per collection
- Best for: GUI-first workflows, quick manual testing
- Collection structure: `{v: 11, id: "...", name: "...", folders: [], requests: [], auth: {...}, headers: [], variables: [], description: ""}`

---

## Bruno Scoped Test Flows (Recommended)

### What are Scoped Flows?

Scoped flows are **self-contained test scenarios** that chain multiple API requests into a numbered sequence. Each flow:
- Automatically handles authentication (login → use token)
- Passes data between steps via environment variables
- Runs independently with isolated environments
- Documents its purpose in `flow.md`

### Flow Directory Structure

```
bruno-collections/{resource-collection}/
├── README.md                          # Collection documentation
├── environments/
│   ├── Local.bru.example             # Template for all flows (reference only)
│   └── Local.bru                     # Parent env (optional, gitignored)
│
├── {flow-folder}/                    # ← Flow directory (kebab-case name)
│   ├── bruno.json                    # Makes it a runnable collection
│   ├── flow.md                       # Documents purpose and steps
│   ├── environments/
│   │   └── Local.bru                 # Flow-specific env (gitignored, real credentials)
│   ├── 1-login.bru                   # Step 1: Always login first
│   ├── 2-create-resource.bru         # Step 2: Create entity
│   ├── 3-get-detail.bru              # Step 3: Verify creation
│   ├── 4-update-resource.bru         # Step 4: Update entity
│   └── 5-get-detail-after-update.bru # Step 5: Verify update
│
├── {another-flow}/                   # Additional flows for this resource
│   ├── bruno.json
│   ├── environments/Local.bru
│   ├── 1-login.bru
│   ├── 2-create-resource.bru
│   ├── 3-delete-resource.bru
│   ├── 4-get-detail-expect-404.bru
│   └── flow.md
│
└── {standalone-request}.bru          # (Optional) standalone for quick testing
```

### Key Design Principles

| Principle | Rationale |
|-----------|-----------|
| Flows are the **primary** testing method | Self-contained flows with auto-auth are easier to run than standalone files |
| Flows live **directly** in the collection root | Easy to find, no nested `flows/` directory |
| Numbered prefixes (`1-`, `2-`, `3-`) | Bruno runs files alphabetically — numbers enforce execution order |
| Login is **duplicated** per flow | Each flow is fully self-contained, no cross-dependencies |
| **Per-flow environments** | Each flow has its own `environments/` for complete isolation |
| `flow.md` per flow folder | Documents the purpose, steps, and expected outcomes |

---

## Workflows

### Creating a New Flow (Bruno)

1. **Create flow folder** with kebab-case name in resource collection root
2. **Create `bruno.json`** to make it a runnable collection
3. **Set up per-flow environment** (`environments/Local.bru` copied from parent's example)
4. **Add `1-login.bru`** (copy from template — identical across all flows)
5. **Write action steps** numbered sequentially (`2-`, `3-`, `4-`, etc.)
6. **Add verification steps** to validate each action
7. **Write `flow.md`** documenting purpose, steps, and expected outcomes
8. **Test run** with `bru run . --env-file environments/Local.bru` from flow directory

See [guidelines/bruno/02-templates.md](guidelines/bruno/02-templates.md) for detailed templates.

### Adding a New Endpoint

1. **Check if flow exists** for the use case
2. **Identify which flow** should include the new endpoint
3. **Create/update request** following numbered sequence
4. **Add response assertions** (status, field values)
5. **Update `flow.md`** if steps change
6. **Validate**: Run flow with `bru run . --env-file environments/Local.bru`

### Creating New CRUD Module

1. **Create collection folder** in `bruno-collections/{resource}/`
2. **Add parent `environments/Local.bru.example`** as template
3. **Create flows** (not standalone files):
   - `create-{resource}/` — Create + Verify
   - `update-{resource}/` — Create + Update + Verify
   - `delete-{resource}/` — Create + Delete + Verify 404
   - `list-{resource}/` — List + Filter + Pagination
4. **Each flow includes**:
   - `1-login.bru` — Authenticate
   - Action steps (`2-`, `3-`, etc.)
   - Verification steps
   - `flow.md` documentation
5. **Configure auth**: `auth: bearer` with `{{accessToken}}`
6. **Test all flows** to verify they work

---

## Pre-Update Checklist

- [ ] Which tool? (Bruno vs Hoppscotch) — **Prefer Bruno flows**
- [ ] New flow or update existing?
- [ ] New endpoint or update existing?
- [ ] Auth type? (public/protected)
- [ ] Need to set environment variables between steps?
- [ ] Response assertions defined?
- [ ] `flow.md` updated?

---

## Common Patterns

### Bruno Flow Patterns

| Pattern | Steps | Use Case |
|---------|-------|----------|
| **Create + Verify** | `1-login → 2-create → 3-get-detail` | Verify creation works |
| **Create + Update + Verify** | `1-login → 2-create → 3-update → 4-get-detail` | Verify updates |
| **Create + Delete + Verify 404** | `1-login → 2-create → 3-delete → 4-get-detail-404` | Verify soft-delete |
| **List + Filter** | `1-login → 2-list-all → 3-list-filtered` | Verify filtering |
| **Full Lifecycle** | `1-login → 2-create → 3-update → 4-delete → 5-verify-404` | Complete CRUD |
| **Multi-Step Domain** | `1-login → 2-create → 3-action → 4-verify` | Complex flows (e.g., refund) |

### Variable Syntax

| Task | Bruno | Hoppscotch |
|------|-------|------------|
| Base URL | `{{baseUrl}}` | `<<baseUrl>>` |
| Variable set | `bru.setVar("id", json.data.id)` | `hopp.env.active.set("id", ...)` |
| Variable reference | `{{entityId}}` | `<<entityId>>` |
| Auth | `auth: bearer` | `"authType": "inherit"` |
| JSON escaping | Not needed | **Single-level only** (`\n` not `\\n`) |

---

## Running Flows

### Run a Specific Flow

```bash
# Navigate to the flow directory
cd bruno-collections/{resource-collection}/{flow-folder}

# Run with the flow's own environment
bru run . --env-file environments/Local.bru
```

### Run with Delay Between Requests

```bash
cd bruno-collections/{resource-collection}/{flow-folder}
bru run . --env-file environments/Local.bru --delay 500
```

### Run with JSON Output

```bash
cd bruno-collections/{resource-collection}/{flow-folder}
bru run . --env-file environments/Local.bru -o results.json
```

---

## References

### Bruno
- [Flow Patterns](guidelines/bruno/01-flow-patterns.md)
- [Templates](guidelines/bruno/02-templates.md)
- [Environment Variables](guidelines/bruno/03-environment-variables.md)
- [Troubleshooting](guidelines/bruno/04-troubleshooting.md)

### Hoppscotch
- [Basics](guidelines/hoppscotch/01-basics.md)

### General
- [Quick Reference](REFERENCE.md)
- [Full Documentation](/docs/knowledge-base/09-bruno-scoped-flows.md)
