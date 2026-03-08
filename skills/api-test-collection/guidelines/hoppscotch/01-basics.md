# Hoppscotch Guidelines

GUI-first API testing tool. Best for manual testing and exploration.

---

## Collection Location

```
hoppscotch-collections/{entity-name}-collection.json
```

---

## Collection Structure

Each collection is a standalone JSON file:

```json
{
  "v": 11,
  "id": "unique-collection-id",
  "name": "Collection Name",
  "folders": [],
  "requests": [...],
  "auth": {
    "authType": "inherit",
    "authActive": true
  },
  "headers": [],
  "variables": [],
  "description": "Collection description"
}
```

---

## Critical: JSON Escaping Rules

**Body content must use single-level escaping only.**

| Character | Correct | Wrong |
|-----------|---------|-------|
| Newline | `\n` | `\\n` |
| Quote | `\"` | `\\\"` |
| Backslash | `\\` | `\\\\` |

**Correct:**
```json
{
  "body": {
    "contentType": "application/json",
    "body": "{\n  \"name\": \"Outlet Name\"\n}"
  }
}
```

**Incorrect (double-escaped):**
```json
{
  "body": {
    "contentType": "application/json",
    "body": "{\\n  \\\"name\\\": \\\"Outlet Name\\\"\\n}"
  }
}
```

---

## Request Structure

```json
{
  "v": "17",
  "name": "Request Name",
  "method": "GET|POST|PATCH|DELETE",
  "endpoint": "<<baseUrl>>/api/v1/...",
  "params": [
    {
      "key": "outletId",
      "value": "<<outletId>>",
      "active": true,
      "description": "Parameter description"
    }
  ],
  "headers": [
    {
      "key": "Content-Type",
      "value": "application/json",
      "active": true,
      "description": ""
    }
  ],
  "preRequestScript": "",
  "testScript": "hopp.env.active.set(\"variableName\", pw.response.body.data.id);",
  "auth": {
    "authType": "inherit",
    "authActive": true
  },
  "body": {
    "contentType": "application/json",
    "body": "{\n  \"field\": \"value\"\n}"
  },
  "requestVariables": [],
  "responses": {
    "Success": {
      "name": "Success",
      "originalRequest": {
        "v": "6",
        "name": "Request Name",
        "method": "GET",
        "endpoint": "<<baseUrl>>/api/v1/...",
        "headers": [...],
        "params": [...],
        "body": {...},
        "auth": {...},
        "requestVariables": []
      },
      "status": "OK",
      "code": 200,
      "headers": [
        {"key": "content-type", "value": "application/json"}
      ],
      "body": "{\"data\":{...}}"
    },
    "Failed": {
      "name": "Failed",
      "originalRequest": {...},
      "status": "Not Found",
      "code": 404,
      "headers": [...],
      "body": "{\"error\":\"Resource not found\"}"
    }
  },
  "description": "Brief description or null"
}
```

---

## Auth Configuration

**Public endpoints:**
```json
{
  "auth": {
    "authType": "none",
    "authActive": false
  }
}
```

**Protected endpoints (inherit from collection):**
```json
{
  "auth": {
    "authType": "inherit",
    "authActive": true
  }
}
```

---

## Test Scripts

**After successful auth:**
```javascript
const res = pw.response.body;
hopp.env.active.set("accessToken", res.data.access_token);
hopp.env.active.set("refreshToken", res.data.refresh_token);
```

**After creating resource:**
```javascript
hopp.env.active.set("outletId", pw.response.body.data.slug);
hopp.env.active.set("serviceId", pw.response.body.data.slug);
```

---

## Validation Command

```bash
# Validate a specific collection
cat hoppscotch-collections/{entity-name}-collection.json | jq . > /dev/null && echo "Valid JSON"

# Validate all collections
for f in hoppscotch-collections/*-collection.json; do
  echo -n "Checking $f... "
  cat "$f" | jq . > /dev/null && echo "Valid" || echo "INVALID"
done
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Double-escaped body | Use `\n` not `\\n`, use `"` not `\"` inside JSON strings |
| Missing response examples | Always add Success + Failed/Error cases |
| Wrong auth type | Public = `none`, Protected = `inherit` |
| Missing `contentType: null` for GET | Set both `contentType` and `body` to `null` for GET requests |
| Hardcoded IDs | Use `<<variableName>>` syntax |
| No test script for auth | Set tokens after sign-in/sign-up |
| Missing collection `id` | Generate unique ID for each collection |
| Wrong `v` version | Use `v: 11` for collections, `v: "17"` for requests |

---

## Variable Syntax

| Concept | Hoppscotch |
|---------|------------|
| Variable reference | `<<baseUrl>>` |
| Set variable | `hopp.env.active.set("id", value)` |
| Get response body | `pw.response.body` |
| Response status | `pw.response.status` |
