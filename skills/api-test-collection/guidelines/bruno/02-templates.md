# Bruno Flow Templates

Copy these templates when creating new flows.

---

## Flow bruno.json

```json
{
  "version": "1",
  "name": "{Flow Name}",
  "type": "collection"
}
```

---

## Flow environments/Local.bru

```bru
vars {
  baseUrl: http://localhost:8787
  testEmail: your-actual-email@example.com
  testPassword: your-actual-password
  accessToken:
  # Add flow-specific vars here (e.g., outletId for outlet-scoped flows)
}
```

---

## 1-login.bru (Required First Step)

Every flow starts with this identical login step:

```bru
meta {
  name: Login
  type: http
  seq: 1
}

post {
  url: {{baseUrl}}/api/v1/auth/sign-in
  body: json
  auth: none
}

body:json {
  {
    "email": "{{testEmail}}",
    "password": "{{testPassword}}"
  }
}

assert {
  res.status: eq 200
  res.body.data.access_token: isDefined
}

script:post-response {
  const json = res.getBody();
  if (json.data && json.data.access_token) {
    bru.setVar("accessToken", json.data.access_token);
  }
}
```

---

## Action Step Template (POST/PATCH/DELETE)

```bru
meta {
  name: {Step Description}
  type: http
  seq: {step number}
}

{method} {
  url: {{baseUrl}}/api/v1/{resource-path}
  body: json
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

body:json {
  {
    "field": "value"
  }
}

assert {
  res.status: eq {expected status code}
  res.body.data.id: isDefined
}

script:post-response {
  const json = res.getBody();
  if (json.data && json.data.id) {
    bru.setVar("{resourceId}", json.data.id);
  }
}
```

**Example - Create Article Type:**
```bru
meta {
  name: Create Article Type (Shared)
  type: http
  seq: 2
}

post {
  url: {{baseUrl}}/api/v1/article-types
  body: json
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

body:json {
  {
    "name": "CRUD Test Shared Type",
    "scope": "shared"
  }
}

assert {
  res.status: eq 201
  res.body.data.id: isDefined
  res.body.data.isShared: eq true
}

script:post-response {
  const json = res.getBody();
  if (json.data && json.data.id) {
    bru.setVar("articleTypeId", json.data.id);
  }
}
```

---

## Verification Step Template (GET)

```bru
meta {
  name: {Verify Description}
  type: http
  seq: {step number}
}

get {
  url: {{baseUrl}}/api/v1/{resource-path}/{{resourceId}}
  body: none
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

assert {
  res.status: eq 200
  res.body.data.id: eq {{resourceId}}
  res.body.data.{field}: eq {expected value}
  res.body.data.deletedAt: isNull
}
```

**Example - Verify Article Type Created:**
```bru
meta {
  name: Verify Article Type Created
  type: http
  seq: 3
}

get {
  url: {{baseUrl}}/api/v1/article-types/{{articleTypeId}}
  body: none
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

assert {
  res.status: eq 200
  res.body.data.id: eq {{articleTypeId}}
  res.body.data.name: eq "CRUD Test Shared Type"
  res.body.data.isShared: eq true
  res.body.data.deletedAt: isNull
}
```

---

## Verification Step (Expect 404 - After Delete)

```bru
meta {
  name: Verify Deleted
  type: http
  seq: {step number}
}

get {
  url: {{baseUrl}}/api/v1/{resource-path}/{{resourceId}}
  body: none
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

assert {
  res.status: eq 404
}
```

**Example - Verify Article Type Soft Deleted:**
```bru
meta {
  name: Verify Article Type Soft Deleted
  type: http
  seq: 5
}

get {
  url: {{baseUrl}}/api/v1/article-types/{{articleTypeId}}
  body: none
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

assert {
  res.status: eq 404
}
```

---

## Update Step Template (PATCH)

```bru
meta {
  name: Update {Resource}
  type: http
  seq: {step number}
}

patch {
  url: {{baseUrl}}/api/v1/{resource-path}/{{resourceId}}
  body: json
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

body:json {
  {
    "name": "Updated {Resource} Name"
  }
}

assert {
  res.status: eq 200
  res.body.data.id: eq {{resourceId}}
  res.body.data.name: eq "Updated {Resource} Name"
}
```

---

## Delete Step Template

```bru
meta {
  name: Delete {Resource} (Soft)
  type: http
  seq: {step number}
}

delete {
  url: {{baseUrl}}/api/v1/{resource-path}/{{resourceId}}
  body: none
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

assert {
  res.status: eq 200
  res.body.message: contains deleted
}
```

---

## flow.md Template

```markdown
# {Flow Name}

## Purpose
{What this flow tests and why it matters}

## Steps
1. **Login** — Authenticate with test credentials
2. **{Action}** — {description}
3. **{Verify}** — {description}

## Prerequisites
- {Any required data that must already exist, e.g., "Valid outletId"}

## Expected Outcome
- {What the end state should be}

## Related Endpoints
- `POST /api/v1/auth/sign-in`
- `POST /api/v1/{resource}`
- `GET /api/v1/{resource}/{id}`
```

**Example:**
```markdown
# Delete Lifecycle - Article Types

## Purpose
Tests the delete/soft-delete lifecycle for an article type.

## Steps
1. **Login** — Authenticate with test credentials
2. **Create Article Type** — POST with scope=shared, saves articleTypeId
3. **Verify Creation** — GET the created article type, verify it exists
4. **Soft Delete** — DELETE the article type, verify success message
5. **Verify Deletion** — GET the article type again, expect 404

## Prerequisites
- Valid test user credentials in environment

## Expected Outcome
- Article type is created successfully
- GET after creation returns the article
- DELETE performs soft delete
- GET after delete returns 404

## Related Endpoints
- `POST /api/v1/auth/sign-in`
- `POST /api/v1/article-types`
- `GET /api/v1/article-types/:id`
- `DELETE /api/v1/article-types/:id`
```

---

## Parent Collection README.md Template

```markdown
# {Entity} API Collection

Brief description of what this collection tests.

## Directory Structure

```
bruno-collections/
├── environments/
│   └── Local.bru.example          # Shared template for ALL collections
│
├── {resource-collection}/
│   ├── README.md
│   ├── {flow-folder}/              # Flow: Create + Verify
│   │   ├── bruno.json
│   │   ├── flow.md
│   │   ├── environments/Local.bru
│   │   ├── 1-login.bru
│   │   ├── 2-create-{resource}.bru
│   │   └── 3-get-detail.bru
│   │
│   ├── {another-flow}/            # Flow: Create + Delete + Verify 404
│   │   ├── bruno.json
│   │   ├── flow.md
│   │   ├── environments/Local.bru
│   │   ├── 1-login.bru
│   │   ├── 2-create-{resource}.bru
│   │   ├── 3-delete-{resource}.bru
│   │   └── 4-get-detail-expect-404.bru
│   │
│   └── (Optional) standalone.bru  # For quick manual testing
```

## Setup

1. **Copy the example environment for each flow:**
   ```bash
   cp bruno-collections/environments/Local.bru.example \
      bruno-collections/{resource-collection}/{flow-folder}/environments/Local.bru
   ```

2. **Edit values in each flow's `environments/Local.bru`:**
   - `baseUrl`: Your API base URL
   - `testEmail`: Valid test user email
   - `testPassword`: Valid test user password

3. **Flows automatically handle authentication** — no manual token needed.

## Running Flows

```bash
# Run a specific flow
cd {flow-folder}
bru run . --env-file environments/Local.bru

# Run with delay
cd {flow-folder}
bru run . --env-file environments/Local.bru --delay 500
```

## Available Flows

| Flow | Pattern | Description |
|------|---------|-------------|
| `{flow-folder}` | Create + Verify | Tests creating {resource} |
| `{another-flow}` | Create + Delete + 404 | Tests soft delete |

## Environment Variables

| Variable | Description | Auto-set |
|----------|-------------|----------|
| `baseUrl` | API base URL | Manual |
| `testEmail` | Test user email | Manual |
| `testPassword` | Test user password | Manual |
| `accessToken` | JWT token | ✓ Auto (by 1-login.bru) |
| `{resource}Id` | Created {resource} ID | ✓ Auto |

## Security Note

- All `environments/Local.bru` files are gitignored
- Never commit real tokens
- Use `Local.bru.example` as template only
```

---

## Complete Flow Example: Create + Verify

**Folder:** `create-article-type/`

**bruno.json:**
```json
{
  "version": "1",
  "name": "Create Article Type Flow",
  "type": "collection"
}
```

**environments/Local.bru:**
```bru
vars {
  baseUrl: http://localhost:8787
  testEmail: test@example.com
  testPassword: password123
  accessToken:
}
```

**1-login.bru:** (Copy from template above)

**2-create-article-type.bru:**
```bru
meta {
  name: Create Article Type (Shared)
  type: http
  seq: 2
}

post {
  url: {{baseUrl}}/api/v1/article-types
  body: json
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

body:json {
  {
    "name": "Test Article Type",
    "scope": "shared"
  }
}

assert {
  res.status: eq 201
  res.body.data.id: isDefined
}

script:post-response {
  const json = res.getBody();
  if (json.data && json.data.id) {
    bru.setVar("articleTypeId", json.data.id);
  }
}
```

**3-get-detail.bru:**
```bru
meta {
  name: Verify Article Type Created
  type: http
  seq: 3
}

get {
  url: {{baseUrl}}/api/v1/article-types/{{articleTypeId}}
  body: none
  auth: bearer
}

auth:bearer {
  token: {{accessToken}}
}

assert {
  res.status: eq 200
  res.body.data.id: eq {{articleTypeId}}
  res.body.data.name: eq "Test Article Type"
}
```

**flow.md:** (Copy from template above)
