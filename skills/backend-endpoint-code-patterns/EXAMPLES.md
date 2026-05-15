# Backend Endpoint Code Pattern Examples

## Good: mount-only `route.ts`

```ts
import { Hono } from 'hono'
import { getRoute } from './services/get'
import { postRoute } from './services/post'
import { patchRoute } from './services/patch'
import { deleteRoute } from './services/delete'
import type { AppEnv } from '@/types/env'

export const resourceRoute = new Hono<AppEnv>()
  .route('/', getRoute)
  .route('/', postRoute)
  .route('/', patchRoute)
  .route('/', deleteRoute)

export type ResourceRoute = typeof resourceRoute
```

## Bad: fat `route.ts`

```ts
resourceRoute.get('/', authMiddleware, async (c) => {
  const db = c.get('db')
  const rows = await db.select().from(resources)
  return c.json({ data: rows })
})
```

Problems:

- DB access inside route handler.
- No repository boundary.
- No DTO projection.
- No query validation.
- Raw DB rows returned.

## Good: service orchestration

```ts
export const getRoute = new Hono<AppEnv>().get(
  '/',
  authMiddleware,
  customValidator('query', ResourceListQueryDTOSchema),
  async (c) => {
    const user = c.get('user')
    const query = c.req.valid('query')
    const runWithRLS = c.get('runWithRLS')

    const result = await findResourceList({ runWithRLS, userId: user.id, query })

    return c.json({
      data: result.data.map(toResourceDTO),
      meta: result.meta,
    })
  },
)
```

## Good: repository data access only

```ts
export async function findResourceById(args: {
  runWithRLS: RunWithRLS
  id: string
}) {
  return args.runWithRLS((tx) =>
    tx.query.resources.findFirst({
      where: and(eq(resources.id, args.id), isNull(resources.deletedAt)),
    }),
  )
}
```

## Bad: repository throwing HTTP-specific errors

```ts
export async function findResourceOrThrow(id: string) {
  const resource = await db.query.resources.findFirst({ where: eq(resources.id, id) })

  if (!resource) {
    throw new NotFoundError('Resource')
  }

  return resource
}
```

Problem: repository now knows HTTP semantics. Throw route-facing errors from services.
