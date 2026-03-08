# Bruno Flow Patterns

Common patterns for scoped test flows. Each pattern solves a specific testing scenario.

---

## Pattern 1: Create + Verify

**Flow:**
```
1-login.bru → 2-create-{resource}.bru → 3-get-detail.bru
```

**Use for:** Verifying creation works and the data is persisted correctly.

**Example:** Testing that creating an article type actually saves it to the database and returns the correct fields.

---

## Pattern 2: Create + Update + Verify

**Flow:**
```
1-login.bru → 2-create-{resource}.bru → 3-update-{resource}.bru → 4-get-detail.bru
```

**Use for:** Verifying updates are applied correctly.

**Example:** Testing that updating an article type's name actually changes it in the database.

---

## Pattern 3: Create + Delete + Verify 404

**Flow:**
```
1-login.bru → 2-create-{resource}.bru → 3-delete-{resource}.bru → 4-get-detail-expect-404.bru
```

**Use for:** Verifying soft-delete behavior.

**Example:** Testing that deleting an article type makes it unretrievable (returns 404).

---

## Pattern 4: List + Filter Verification

**Flow:**
```
1-login.bru → 2-list-all.bru → 3-list-with-search.bru → 4-list-with-pagination.bru
```

**Use for:** Verifying list endpoints with various filter combinations.

**Example:** Testing that list endpoints support search, pagination, and filters correctly.

---

## Pattern 5: Multi-Step Lifecycle

**Flow:**
```
1-login.bru → 2-create.bru → 3-pay.bru → 4-verify-payment.bru → 5-update-status.bru → 6-get-detail.bru
```

**Use for:** Complex domain flows like transaction lifecycle, refund flow.

**Example:** Testing a complete transaction flow: create → process payment → verify → update status.

---

## Pattern 6: Dependent Resources (Setup → Action → Verify)

**Flow:**
```
1-login.bru → 2-create-parent.bru → 3-create-child.bru → 4-get-child-detail.bru
```

**Use for:** Resources that require a parent to exist first (e.g., delivery needs a transaction).

**Example:** Testing that you can create a delivery only after creating a transaction.

---

## Pattern 7: Error/Edge Case Verification

**Flow (with login):**
```
1-login.bru → 2-create-{resource}.bru → 3-duplicate-create-expect-409.bru
```

**Flow (without login - testing unauthorized access):**
```
1-get-without-auth-expect-401.bru
```

**Use for:** Verifying error handling and edge cases.

**Examples:**
- Testing that creating duplicate resources returns 409
- Testing that accessing protected endpoints without auth returns 401
- Testing validation errors return 400 with proper messages

---

## Choosing the Right Pattern

| Scenario | Pattern |
|----------|---------|
| Just testing create works | Pattern 1: Create + Verify |
| Testing create and update | Pattern 2: Create + Update + Verify |
| Testing soft delete | Pattern 3: Create + Delete + Verify 404 |
| Testing list endpoints | Pattern 4: List + Filter Verification |
| Complex business flow | Pattern 5: Multi-Step Lifecycle |
| Child resources | Pattern 6: Dependent Resources |
| Error handling | Pattern 7: Error/Edge Case |
| Full CRUD coverage | Combine patterns 1, 2, and 3 |

---

## Combining Patterns

You can combine multiple patterns for comprehensive coverage:

**Full CRUD Lifecycle:**
```
1-login → 2-create → 3-verify-create → 4-update → 5-verify-update → 6-delete → 7-verify-404
```

**Create + List Verification:**
```
1-login → 2-create → 3-list-all (verify new item appears) → 4-list-filtered (verify filtering works)
```

**Complex Domain Flow:**
```
1-login → 2-create-transaction → 3-create-refund → 4-approve-refund → 5-verify-refund-status
```

---

## Naming Conventions for Flow Folders

| Pattern | Suggested Name |
|---------|----------------|
| Create + Verify | `create-{resource}` |
| Create + Update + Verify | `update-{resource}` |
| Create + Delete + Verify 404 | `delete-{resource}` or `soft-delete-{resource}` |
| List + Filter | `list-{resource}` |
| Multi-Step | `{domain}-lifecycle` (e.g., `transaction-lifecycle`) |
| Dependent Resources | `create-{child}-with-{parent}` |
| Error Cases | `{resource}-validation` or `{resource}-errors` |
| Full CRUD | `{resource}-full-lifecycle` |
