# Troubleshooting

Common issues and solutions when working with Bruno test flows.

---

## Login fails with 401

**Symptoms:**
- `1-login.bru` returns 401 Unauthorized
- Error: "Invalid login credentials"

**Solutions:**

1. **Verify credentials in `environments/Local.bru`:**
   ```bru
   vars {
     testEmail: your-actual-email@example.com
     testPassword: your-actual-password
   }
   ```

2. **Check `.env.local` for correct credentials:**
   ```bash
   cat .env.local | grep TEST_
   ```

3. **Ensure the test user exists in Supabase Auth**

4. **Verify the user is confirmed** (not pending email verification)

---

## Variable not set between steps

**Symptoms:**
- Later steps fail with "variable not found"
- URL shows `{{articleTypeId}}` instead of actual ID

**Solutions:**

1. **Check the `script:post-response` block in the producing step:**
   ```javascript
   script:post-response {
     const json = res.getBody();
     if (json.data && json.data.id) {
       bru.setVar("articleTypeId", json.data.id);
     }
   }
   ```

2. **Verify the response JSON path:**
   - Is it `json.data.id` or `json.data.slug`?
   - Run the step individually and check the response body

3. **Check for response errors:**
   - If the create step failed (4xx/5xx), the script won't run
   - Fix the create step first

4. **Verify variable name consistency:**
   - Creating step: `bru.setVar("articleTypeId", ...)`
   - Using step: `{{articleTypeId}}`
   - Must match exactly (case-sensitive)

---

## Steps run out of order

**Symptoms:**
- Flow executes files in wrong sequence
- Dependencies fail because earlier step hasn't run

**Solutions:**

1. **Verify numbered prefixes:**
   ```
   1-login.bru
   2-create-article-type.bru
   3-get-detail.bru
   ```

2. **Check `seq` values in `meta` blocks:**
   ```bru
   meta {
     name: Create Article Type
     type: http
     seq: 2  // Must match file prefix
   }
   ```

3. **Remember:** Bruno sorts alphabetically — numbers enforce order

---

## "Collection not found" error

**Symptoms:**
- `bru run` fails with "collection not found"
- Can't run the flow

**Solutions:**

1. **Ensure `bruno.json` exists in the flow folder**

2. **Verify `bruno.json` content:**
   ```json
   {
     "version": "1",
     "name": "Flow Name",
     "type": "collection"
   }
   ```

3. **Run from inside the flow directory:**
   ```bash
   cd bruno-collections/{resource}/{flow-folder}
   bru run . --env-file environments/Local.bru
   ```

---

## Assert failures

**Symptoms:**
- Assertion fails but response looks correct
- Status code assertions fail unexpectedly

**Solutions:**

1. **Run the step individually to see the actual response:**
   ```bash
   bru run --env-file environments/Local.bru "3-get-detail.bru"
   ```

2. **Check if expected value needs quotes:**
   ```bru
   // String values need quotes
   res.body.data.name: eq "Test Name"
   
   // Boolean/number values don't
   res.body.data.isShared: eq true
   res.body.data.count: eq 5
   ```

3. **For dynamic values like IDs, use `isDefined`:**
   ```bru
   res.body.data.id: isDefined  // Instead of exact match
   ```

4. **Check data types:**
   ```bru
   res.body.data.total: isNumber
   res.body.data.name: isString
   res.body.data.items: isArray
   ```

---

## Token expires mid-flow

**Symptoms:**
- First few steps succeed, later steps fail with 401

**Solutions:**

1. **Flows should complete quickly** — tokens typically last 15+ minutes

2. **For long-running flows, add token refresh logic:**
   ```javascript
   script:post-response {
     if (res.getStatus() === 401) {
       // Re-run login or refresh token
     }
   }
   ```

3. **Check token expiration time** in your auth configuration

---

## Environment file not found

**Symptoms:**
- Error: "environment file not found"
- Variables are undefined

**Solutions:**

1. **Ensure `environments/` directory exists in the flow folder**

2. **Verify file path:**
   ```bash
   ls -la environments/
   # Should show: Local.bru
   ```

3. **Run from the correct directory:**
   ```bash
   cd bruno-collections/{resource}/{flow-folder}
   bru run . --env-file environments/Local.bru
   ```

---

## Request body issues

**Symptoms:**
- 400 Bad Request
- Validation errors
- "Unexpected token" errors

**Solutions:**

1. **Verify JSON syntax:**
   ```bru
   body:json {
     {
       "name": "Test Name",
       "scope": "shared"
     }
   }
   ```

2. **Check required fields:**
   - Review API documentation
   - Compare with working requests

3. **Check data types:**
   - Strings need quotes: `"123"`
   - Numbers don't: `123`
   - Booleans: `true` or `false`

4. **Check for trailing commas:**
   ```json
   // Bad:
   {
     "name": "Test",
   }
   
   // Good:
   {
     "name": "Test"
   }
   ```

---

## Common Mistakes Reference

| Mistake | Fix |
|---------|-----|
| No numbered file prefixes | Use `1-`, `2-`, `3-` prefixes to control execution order |
| Missing `1-login.bru` | Every flow must start with login for auto-auth |
| Shared environment between flows | Each flow should have its own `environments/` directory |
| No `flow.md` | Document the flow's purpose, steps, and expected outcomes |
| Missing `script:post-response` | Add to create request to auto-set IDs for later steps |
| Hardcoded IDs in requests | Always use `{{variable}}` syntax |
| No `.gitignore` for environments | Add immediately to prevent token leaks |
| Wrong variable name case | Variable names are case-sensitive: `articleTypeId` ≠ `articletypeid` |
| Missing `auth: bearer` | Protected endpoints need bearer auth with `{{accessToken}}` |
| Wrong `seq` number | `seq` in meta block must match file prefix number |

---

## Debug Mode

Run with verbose output to debug issues:

```bash
bru run . --env-file environments/Local.bru --verbose
```

Or run individual steps:

```bash
bru run --env-file environments/Local.bru "2-create-article-type.bru"
```

---

## Getting Help

If issues persist:

1. Check the [Bruno documentation](https://docs.usebruno.com/)
2. Review the flow's `flow.md` for expected behavior
3. Compare with working flows in the same collection
4. Check the main project knowledge base: `docs/knowledge-base/09-bruno-scoped-flows.md`
