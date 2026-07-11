---
name: api-test-collection
description: Create and maintain API test collections. Use Bruno for scoped flow tests and Hoppscotch for standalone collection tests. Both tools serve different purposes and are equally important.
---

# API Test Collection Skill

## Quick Start

**Both tools serve different purposes and are equally important:**

**Bruno** — Primary for **Scoped Flow Tests**:
- Location: `bruno-collections/{resource-collection}/`
- Format: `.bru` files in numbered flow folders
- Best for: Multi-step flows, CI/CD automation, version control, sequential request chains
- **Key concept**: Scoped flows that chain requests with auto-authentication
- **Auth-first rule**: If any request needs access tokens, refresh tokens, bearer auth, cookies, sessions, API keys derived from auth, or any authentication state, create a login/auth step first before the protected case.
- **Feature-dependency rule**: Before creating a test case for an endpoint, analyze the feature workflow and add required setup/verification steps. Do not test protected or nested resources in isolation when they require parent entities, permissions, seed state, or downstream validation.

**Hoppscotch** — Primary for **Standalone Collection Tests**:
- Location: `hoppscotch-collections/{entity-name}-collection.json`
- Format: Single JSON file per collection
- Best for: GUI-first workflows, quick manual testing, standalone request organization
- Collection structure: `{v: 11, id: "...", name: "...", folders: [], requests: [], auth: {...}, headers: [], variables: [], description: ""}`

---

## Bruno — Scoped Test Flows

### What are Scoped Flows?

Scoped flows are **self-contained test scenarios** that chain multiple API requests into a numbered sequence. Each flow:
- Automatically handles authentication (login → use token)
- Passes data between steps via environment variables
- Runs independently with isolated environments
- Documents its purpose in `flow.md`

### Flow Directory Structure

```
bruno-collections/
├── {resource-collection}/            # ← e.g., "transactions", "contacts"
│   ├── README.md                     # Collection documentation
│   ├── {flow-folder}/                # ← Flow directory (kebab-case name)
│   │   ├── bruno.json                # Makes it a runnable collection
│   │   ├── flow.md                   # Documents purpose and steps
│   │   ├── environments/
│   │   │   └── Local.bru             # Flow-specific env (gitignored, real credentials)
│   │   ├── 1-login.bru               # Step 1: Always login first
│   │   ├── 2-create-resource.bru     # Step 2: Create entity
│   │   ├── 3-get-detail.bru          # Step 3: Verify creation
│   │   ├── 4-update-resource.bru     # Step 4: Update entity
│   │   └── 5-get-detail-after-update.bru # Step 5: Verify update
│   │
│   ├── {another-flow}/               # Additional flows for this resource
│   │   ├── bruno.json
│   │   ├── environments/Local.bru
│   │   ├── 1-login.bru
│   │   ├── 2-create-resource.bru
│   │   ├── 3-delete-resource.bru
│   │   ├── 4-get-detail-expect-404.bru
│   │   └── flow.md
│   │
│   └── {standalone-request}.bru      # (Optional) standalone for quick testing
```

### Key Design Principles

| Principle | Rationale |
|-----------|-----------|
| Flows are the **primary** testing method | Self-contained flows with auto-auth are easier to run than standalone files |
| Flows live **directly** in the collection root | Easy to find, no nested `flows/` directory |
| No parent `bruno.json` by default | Resource root is organizer/docs only; only flow folders need `bruno.json` |
| Numbered prefixes (`1-`, `2-`, `3-`) | Bruno runs files alphabetically — numbers enforce execution order |
| Login is **duplicated** per protected flow | Each auth-dependent flow is fully self-contained, no cross-dependencies |
| Auth dependency triggers login first | If the case requires access/refresh token, bearer token, session cookie, authenticated API key, or auth-derived state, add `1-login.bru` before testing the endpoint |
| Feature dependencies are part of the case | If an endpoint needs parent resources, roles, prior records, uploaded files, or cleanup, include those steps in the same flow before the endpoint under test |
| Verification follows mutation | After create/update/delete actions, add a read/list/detail step that proves the state changed, not only status assertions |
| **Per-flow environments** | Each flow has its own `environments/` for complete isolation |
| `flow.md` per flow folder | Documents the purpose, steps, and expected outcomes |

---

## Workflows

### When to Use Which Tool

| Use Case | Tool | Reason |
|----------|------|--------|
| Multi-step sequential flows | **Bruno** | Numbered execution order, environment variable passing |
| CI/CD integration | **Bruno** | CLI-based, file-based, version control friendly |
| Automated regression testing | **Bruno** | Scriptable, assertions, can run in pipelines |
| API exploration, manual testing | **Hoppscotch** | GUI interface, quick iteration |
| Organizing standalone requests | **Hoppscotch** | Folder structure, visual organization |
| One-off request testing | **Hoppscotch** | No setup, immediate execution |
| Team sharing (non-devs) | **Hoppscotch** | Visual interface, import/export JSON |

### Analyze Feature Dependencies Before Creating Cases

When asked to create a test case for an API endpoint, do not start from the endpoint alone. First inspect the feature and decide whether the endpoint can be tested directly or needs a larger flow.

Ask:
- Does it require authentication or auth-derived state? Add `1-login.bru` first.
- Does it require a parent entity? Create it first in the same flow.
- Does it require a role or permission? Create/assign that member role before the protected action.
- Does it require existing child data, uploaded files, categories, transactions, or external state? Set that state up first.
- How will the result be proven? Add list/detail/read-back validation after mutations.
- Does it create data that should be cleaned up? Add cleanup steps at the end.

Example category create flow:

```text
1-login.bru
2-create-workspace.bru
3-create-category.bru
4-list-categories.bru     # validate category exists
5-update-category.bru
6-delete-category.bru
7-delete-workspace.bru    # cleanup parent resource
```

### Creating a New Flow (Bruno)

1. **Analyze feature dependencies** and identify setup, action, verification, and cleanup steps
2. **Create flow folder** with kebab-case name in resource collection root
3. **Create flow-level `bruno.json`** to make that flow runnable
4. **Set up per-flow environment** (`environments/Local.bru` inside each flow)
5. **Check auth dependency before writing endpoint requests**
   - If the case needs access token, refresh token, bearer auth, cookie session, authenticated API key, or auth-derived state: create `1-login.bru` first.
   - If the repo has a login endpoint: call it and save tokens with `bru.setVar(...)`.
   - If the repo has no login endpoint: create an explicit auth bootstrap step that validates/promotes env-provided tokens, and document the exception in `flow.md`.
   - If the case intentionally tests unauthenticated/public behavior: do not add login; document that it is public/negative-auth.
6. **Write setup steps** before the endpoint under test when dependencies exist
7. **Write action steps** numbered sequentially (`2-`, `3-`, `4-`, etc.)
8. **Add verification steps** to validate each action
9. **Add cleanup steps** when the flow creates parent/test data
10. **Write `flow.md`** documenting purpose, auth setup, dependencies, steps, and expected outcomes
11. **Test run** with `bru run . --env-file environments/Local.bru` from flow directory

See [guidelines/bruno/02-templates.md](guidelines/bruno/02-templates.md) for detailed templates.

### Adding a New Endpoint

1. **Analyze the feature dependency chain** for that endpoint
2. **Check if a flow already covers the required setup/action/verification path**
3. **Identify which flow** should include the new endpoint, or create a new flow if it needs a distinct setup
4. **Add missing setup requests first** (login, parent resource, role, seed data, upload, etc.)
5. **Create/update request** following numbered sequence
6. **Add read-back verification** with list/detail/read-url requests after mutations
7. **Add cleanup** for data created only for the test
8. **Update `flow.md`** if steps, dependencies, or expected outcomes change
9. **Validate**: Run flow with `bru run . --env-file environments/Local.bru`

### Creating New CRUD Module

1. **Create collection folder** in `bruno-collections/{resource}/`
2. **Create flows** (not standalone files):
   - `create-{resource}/` — Setup + Create + Verify
   - `update-{resource}/` — Setup + Create + Update + Verify
   - `delete-{resource}/` — Setup + Create + Delete + Verify 404
   - `list-{resource}/` — Setup + List + Filter + Pagination
3. **Each protected flow includes**:
   - `1-login.bru` — Authenticate/bootstrap tokens
   - Setup steps for parent resources and permissions
   - Action steps (`2-`, `3-`, etc.)
   - Verification steps
   - Cleanup steps when needed
   - `flow.md` documentation
4. **Configure auth**: `auth: bearer` with `{{accessToken}}`
5. **Test all flows** to verify they work

Do not add `bruno.json` to `bruno-collections/{resource}/` unless that root folder itself is intentionally runnable.
Do not add root-level `environments/` for organizer collections.
Do not add root-level or flow-level `environments/Local.bru.example` for organizer collections.

---

## Pre-Update Checklist

- [ ] Which tool fits the use case?
  - **Use Bruno**: Multi-step flows, CI/CD, automation, sequential chains
  - **Use Hoppscotch**: Standalone requests, GUI testing, quick exploration
- [ ] New flow or update existing?
- [ ] New endpoint or update existing?
- [ ] Auth type? (public/protected)
- [ ] If protected, did you create `1-login.bru` before endpoint requests?
- [ ] Does login save every needed auth value (`accessToken`, `refreshToken`, cookies, auth-derived IDs)?
- [ ] If no login endpoint exists, did you add an auth bootstrap step and document the exception?
- [ ] What parent resources are required before the endpoint can work?
- [ ] What roles/permissions or seed data are required?
- [ ] Can this endpoint be tested directly, or does it require a larger flow?
- [ ] Is there a read/list/detail verification after each mutation?
- [ ] Is cleanup included for created test data?
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

### Bruno — Scoped Test Flows
- [Flow Patterns](guidelines/bruno/01-flow-patterns.md)
- [Templates](guidelines/bruno/02-templates.md)
- [Environment Variables](guidelines/bruno/03-environment-variables.md)
- [Troubleshooting](guidelines/bruno/04-troubleshooting.md)

### Hoppscotch — Standalone Collection Tests
- [Basics](guidelines/hoppscotch/01-basics.md)

### General
- [Quick Reference](REFERENCE.md)
- [Full Documentation](/docs/knowledge-base/09-bruno-scoped-flows.md)
