# Environment Variables

Catalog of environment variables used in test flows.

---

## Always Required

These variables must be defined in every flow's `environments/Local.bru`.

When the repo already contains a known-good `Local.bru`, prefer copying those values into new flow environments instead of inventing placeholders.

| Variable | Description | Source | Example |
|----------|-------------|--------|---------|
| `baseUrl` | API base URL | Manual | `http://localhost:8787` |
| `testEmail` | Test user email | `.env.local` → `TEST_EMAIL` | `test@example.com` |
| `testPassword` | Test user password | `.env.local` → `TEST_PASSWORD` | `password123` |
| `accessToken` | JWT token | **Auto-set** by `1-login.bru` | Leave empty |

---

## Resource-Specific (Add as Needed)

Add these based on the resource you're testing:

### Common IDs

| Variable | Used By | Description |
|----------|---------|-------------|
| `outletId` | Most resources | Outlet ID or slug |
| `contactId` | contacts, transactions | Contact UUID |
| `washServiceId` | wash-services, transactions | Wash service UUID |
| `washServiceCategoryId` | wash-service-categories | Category UUID |
| `perfumeId` | perfumes, transactions | Perfume UUID |
| `paymentMethodId` | payment-methods, transactions | Payment method UUID |
| `voucherCode` | vouchers, transactions | Voucher code string |
| `driverId` | transactions (deliveries) | Driver profile ID |
| `termsConditionsId` | terms-conditions | Auto-set by create step |

### Article Types

| Variable | Description |
|----------|-------------|
| `articleTypeId` | Auto-set by create step |

### Transactions

| Variable | Description |
|----------|-------------|
| `transactionId` | Auto-set by create step |
| `deliveryId` | Auto-set by create delivery step |
| `refundId` | Auto-set by create refund step |

### Inventories

| Variable | Description |
|----------|-------------|
| `inventoryId` | Auto-set by create step |

---

## How Variables Are Set

### Manual Variables

Set these in `environments/Local.bru` before running flows:

```bru
vars {
  baseUrl: http://localhost:8787
  testEmail: lc002lucidchart@gmail.com
  testPassword: qweasd123
  accessToken:
  outletId: outlet-123
  contactId: uuid-here
}
```

### Auto-Set Variables

These are set automatically by `script:post-response` blocks:

**After Login:**
```javascript
script:post-response {
  const json = res.getBody();
  if (json.data && json.data.access_token) {
    bru.setVar("accessToken", json.data.access_token);
  }
}
```

**After Creating Resource:**
```javascript
script:post-response {
  const json = res.getBody();
  if (json.data && json.data.id) {
    bru.setVar("articleTypeId", json.data.id);
  }
}
```

---

## Variable Naming Conventions

| Resource | Variable Name | Example |
|----------|--------------|---------|
| article-types | `articleTypeId` | `articleTypeId` |
| transactions | `transactionId` | `transactionId` |
| contacts | `contactId` | `contactId` |
| wash-services | `washServiceId` | `washServiceId` |
| outlets | `outletId` | `outletId` |
| users | `userId` | `userId` |

**Rule:** Use camelCase with the resource name + `Id` suffix.

---

## Parent Collection Template

The parent collection's `environments/Local.bru.example` should list all possible variables:

```bru
vars {
  # Always required
  baseUrl: http://localhost:8787
  testEmail: your-test-email@example.com
  testPassword: your-test-password
  accessToken:
  
  # Resource-specific (fill in as needed)
  outletId: your-outlet-id
  contactId: your-contact-uuid
  washServiceId: your-wash-service-uuid
  paymentMethodId: your-payment-method-uuid
  
  # Auto-set by flows (leave empty)
  articleTypeId:
  transactionId:
  inventoryId:
}
```

Each flow copies this template and fills in only the variables it needs.

Avoid adding flow-level `.gitignore` files when a global gitignore rule already excludes `environments/Local.bru`.
