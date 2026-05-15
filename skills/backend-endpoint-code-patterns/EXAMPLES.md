# Backend Endpoint Code Pattern Complete Example

## Resource: `organizations`

## `src/routes/organizations/route.ts`

```ts
import { Hono } from 'hono'
import { getRoute } from './services/get'
import { postRoute } from './services/post'
import { patchRoute } from './services/patch'
import { deleteRoute } from './services/delete'
import type { AppEnv } from '@/types/env'

export const organizationsRoute = new Hono<AppEnv>()
  .route('/', getRoute)
  .route('/', postRoute)
  .route('/', patchRoute)
  .route('/', deleteRoute)

export type OrganizationsRoute = typeof organizationsRoute
```

## `src/routes/organizations/types/query.ts`

```ts
import { z } from 'zod'

export const OrganizationListQueryDTOSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
  search: z.string().trim().optional(),
  sortBy: z.enum(['name', 'createdAt']).default('createdAt'),
  sortDirection: z.enum(['asc', 'desc']).default('desc'),
})

export const OrganizationDetailParamsDTOSchema = z.object({
  idOrSlug: z.string().min(1),
})

export type OrganizationListQueryDTO = z.infer<typeof OrganizationListQueryDTOSchema>
export type OrganizationDetailParamsDTO = z.infer<typeof OrganizationDetailParamsDTOSchema>
```

## `src/routes/organizations/types/create.ts`

```ts
import { z } from 'zod'

export const CreateOrganizationRequestBodySchema = z.object({
  name: z.string().min(1),
  slug: z.string().min(1),
  description: z.string().optional(),
})

export type CreateOrganizationRequestBody = z.infer<typeof CreateOrganizationRequestBodySchema>
```

## `src/routes/organizations/types/update.ts`

```ts
import { z } from 'zod'

export const UpdateOrganizationRequestBodySchema = z.object({
  name: z.string().min(1).optional(),
  description: z.string().nullable().optional(),
})

export type UpdateOrganizationRequestBody = z.infer<typeof UpdateOrganizationRequestBodySchema>
```

## `src/routes/organizations/types/dto.ts`

```ts
export type OrganizationDTO = {
  id: string
  name: string
  slug: string
  description: string | null
  createdAt: string
  updatedAt: string
}
```

## `src/routes/organizations/lib/mappers.ts`

```ts
import type { OrganizationDTO } from '../types/dto'

type OrganizationRow = {
  id: string
  name: string
  slug: string
  description: string | null
  createdAt: Date
  updatedAt: Date
}

export function toOrganizationDTO(row: OrganizationRow): OrganizationDTO {
  return {
    id: row.id,
    name: row.name,
    slug: row.slug,
    description: row.description,
    createdAt: row.createdAt.toISOString(),
    updatedAt: row.updatedAt.toISOString(),
  }
}
```

## `src/routes/organizations/repositories/organizations.ts`

```ts
import { and, asc, desc, eq, ilike, isNull, sql } from 'drizzle-orm'
import { organizations } from '@/db/schemas'
import type { RunWithRLS } from '@/types/rls'
import type { OrganizationListQueryDTO } from '../types/query'
import type { CreateOrganizationRequestBody } from '../types/create'
import type { UpdateOrganizationRequestBody } from '../types/update'

export async function findOrganizationList(args: {
  runWithRLS: RunWithRLS
  userId: string
  query: OrganizationListQueryDTO
}) {
  const offset = (args.query.page - 1) * args.query.limit
  const orderByName = args.query.sortDirection === 'asc' ? asc(organizations.name) : desc(organizations.name)
  const orderByCreatedAt = args.query.sortDirection === 'asc' ? asc(organizations.createdAt) : desc(organizations.createdAt)
  const orderBy = args.query.sortBy === 'name' ? orderByName : orderByCreatedAt

  const baseWhere = and(
    eq(organizations.createdBy, args.userId),
    isNull(organizations.deletedAt),
    args.query.search ? ilike(organizations.name, `%${args.query.search}%`) : undefined,
  )

  const [rows, totalRows] = await args.runWithRLS(async (tx) => {
    const rowsValue = await tx
      .select()
      .from(organizations)
      .where(baseWhere)
      .orderBy(orderBy, desc(organizations.id))
      .limit(args.query.limit)
      .offset(offset)

    const totalValue = await tx
      .select({ total: sql<number>`count(*)` })
      .from(organizations)
      .where(baseWhere)

    return [rowsValue, totalValue] as const
  })

  const total = Number(totalRows[0]?.total ?? 0)

  return {
    data: rows,
    meta: {
      page: args.query.page,
      limit: args.query.limit,
      total,
      totalPages: Math.max(1, Math.ceil(total / args.query.limit)),
    },
  }
}

export async function findOrganizationById(args: {
  runWithRLS: RunWithRLS
  id: string
  userId: string
}) {
  return args.runWithRLS((tx) =>
    tx.query.organizations.findFirst({
      where: and(
        eq(organizations.id, args.id),
        eq(organizations.createdBy, args.userId),
        isNull(organizations.deletedAt),
      ),
    }),
  )
}

export async function findOrganizationBySlug(args: {
  runWithRLS: RunWithRLS
  slug: string
  userId: string
}) {
  return args.runWithRLS((tx) =>
    tx.query.organizations.findFirst({
      where: and(
        eq(organizations.slug, args.slug),
        eq(organizations.createdBy, args.userId),
        isNull(organizations.deletedAt),
      ),
    }),
  )
}

export async function createOrganization(args: {
  runWithRLS: RunWithRLS
  userId: string
  body: CreateOrganizationRequestBody
}) {
  const [row] = await args.runWithRLS((tx) =>
    tx
      .insert(organizations)
      .values({
        name: args.body.name,
        slug: args.body.slug,
        description: args.body.description ?? null,
        createdBy: args.userId,
      })
      .returning(),
  )

  return row
}

export async function updateOrganization(args: {
  runWithRLS: RunWithRLS
  id: string
  userId: string
  body: UpdateOrganizationRequestBody
}) {
  const [row] = await args.runWithRLS((tx) =>
    tx
      .update(organizations)
      .set({
        ...(args.body.name !== undefined ? { name: args.body.name } : {}),
        ...(args.body.description !== undefined ? { description: args.body.description } : {}),
      })
      .where(
        and(
          eq(organizations.id, args.id),
          eq(organizations.createdBy, args.userId),
          isNull(organizations.deletedAt),
        ),
      )
      .returning(),
  )

  return row
}

export async function softDeleteOrganization(args: {
  runWithRLS: RunWithRLS
  id: string
  userId: string
}) {
  const [row] = await args.runWithRLS((tx) =>
    tx
      .update(organizations)
      .set({ deletedAt: new Date() })
      .where(
        and(
          eq(organizations.id, args.id),
          eq(organizations.createdBy, args.userId),
          isNull(organizations.deletedAt),
        ),
      )
      .returning(),
  )

  return row
}
```

## `src/routes/organizations/lib/resolve-organization.ts`

```ts
import { NotFoundError } from '@/lib/errors'
import { findOrganizationById, findOrganizationBySlug } from '../repositories/organizations'
import type { RunWithRLS } from '@/types/rls'

const UUID_REGEX =
  /^[0-9a-f]{8}-[0-9a-f]{4}-[1-8][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i

export async function resolveOrganizationByIdOrSlug(args: {
  runWithRLS: RunWithRLS
  idOrSlug: string
  userId: string
}) {
  const byId = UUID_REGEX.test(args.idOrSlug)
    ? await findOrganizationById({ runWithRLS: args.runWithRLS, id: args.idOrSlug, userId: args.userId })
    : null

  const bySlug = byId
    ? null
    : await findOrganizationBySlug({ runWithRLS: args.runWithRLS, slug: args.idOrSlug, userId: args.userId })

  const row = byId ?? bySlug

  if (!row) {
    throw new NotFoundError('Organization')
  }

  return row
}
```

## `src/routes/organizations/services/get.ts`

```ts
import { Hono } from 'hono'
import { authMiddleware } from '@/middlewares/auth'
import { customValidator } from '@/middlewares/custom-validator'
import type { AppEnv } from '@/types/env'
import {
  OrganizationDetailParamsDTOSchema,
  OrganizationListQueryDTOSchema,
} from '../types/query'
import { findOrganizationList } from '../repositories/organizations'
import { resolveOrganizationByIdOrSlug } from '../lib/resolve-organization'
import { toOrganizationDTO } from '../lib/mappers'

export const getRoute = new Hono<AppEnv>()
  .get('/', authMiddleware, customValidator('query', OrganizationListQueryDTOSchema), async (c) => {
    const runWithRLS = c.get('runWithRLS')
    const user = c.get('user')
    const query = c.req.valid('query')

    const result = await findOrganizationList({
      runWithRLS,
      userId: user.id,
      query,
    })

    return c.json({
      data: result.data.map(toOrganizationDTO),
      meta: result.meta,
    })
  })
  .get(
    '/:idOrSlug',
    authMiddleware,
    customValidator('param', OrganizationDetailParamsDTOSchema),
    async (c) => {
      const runWithRLS = c.get('runWithRLS')
      const user = c.get('user')
      const { idOrSlug } = c.req.valid('param')

      const row = await resolveOrganizationByIdOrSlug({
        runWithRLS,
        userId: user.id,
        idOrSlug,
      })

      return c.json({ data: toOrganizationDTO(row) })
    },
  )
```

## `src/routes/organizations/services/post.ts`

```ts
import { Hono } from 'hono'
import { authMiddleware } from '@/middlewares/auth'
import { customValidator } from '@/middlewares/custom-validator'
import type { AppEnv } from '@/types/env'
import { CreateOrganizationRequestBodySchema } from '../types/create'
import { createOrganization } from '../repositories/organizations'
import { toOrganizationDTO } from '../lib/mappers'

export const postRoute = new Hono<AppEnv>().post(
  '/',
  authMiddleware,
  customValidator('json', CreateOrganizationRequestBodySchema),
  async (c) => {
    const runWithRLS = c.get('runWithRLS')
    const user = c.get('user')
    const body = c.req.valid('json')

    const row = await createOrganization({
      runWithRLS,
      userId: user.id,
      body,
    })

    return c.json({ data: toOrganizationDTO(row) }, 201)
  },
)
```

## `src/routes/organizations/services/patch.ts`

```ts
import { Hono } from 'hono'
import { NotFoundError } from '@/lib/errors'
import { authMiddleware } from '@/middlewares/auth'
import { customValidator } from '@/middlewares/custom-validator'
import type { AppEnv } from '@/types/env'
import { UpdateOrganizationRequestBodySchema } from '../types/update'
import { OrganizationDetailParamsDTOSchema } from '../types/query'
import { updateOrganization } from '../repositories/organizations'
import { resolveOrganizationByIdOrSlug } from '../lib/resolve-organization'
import { toOrganizationDTO } from '../lib/mappers'

export const patchRoute = new Hono<AppEnv>().patch(
  '/:idOrSlug',
  authMiddleware,
  customValidator('param', OrganizationDetailParamsDTOSchema),
  customValidator('json', UpdateOrganizationRequestBodySchema),
  async (c) => {
    const runWithRLS = c.get('runWithRLS')
    const user = c.get('user')
    const { idOrSlug } = c.req.valid('param')
    const body = c.req.valid('json')

    const current = await resolveOrganizationByIdOrSlug({
      runWithRLS,
      userId: user.id,
      idOrSlug,
    })

    const row = await updateOrganization({
      runWithRLS,
      id: current.id,
      userId: user.id,
      body,
    })

    if (!row) {
      throw new NotFoundError('Organization')
    }

    return c.json({ data: toOrganizationDTO(row) })
  },
)
```

## `src/routes/organizations/services/delete.ts`

```ts
import { Hono } from 'hono'
import { NotFoundError } from '@/lib/errors'
import { authMiddleware } from '@/middlewares/auth'
import { customValidator } from '@/middlewares/custom-validator'
import type { AppEnv } from '@/types/env'
import { OrganizationDetailParamsDTOSchema } from '../types/query'
import { resolveOrganizationByIdOrSlug } from '../lib/resolve-organization'
import { softDeleteOrganization } from '../repositories/organizations'

export const deleteRoute = new Hono<AppEnv>().delete(
  '/:idOrSlug',
  authMiddleware,
  customValidator('param', OrganizationDetailParamsDTOSchema),
  async (c) => {
    const runWithRLS = c.get('runWithRLS')
    const user = c.get('user')
    const { idOrSlug } = c.req.valid('param')

    const current = await resolveOrganizationByIdOrSlug({
      runWithRLS,
      userId: user.id,
      idOrSlug,
    })

    const row = await softDeleteOrganization({
      runWithRLS,
      id: current.id,
      userId: user.id,
    })

    if (!row) {
      throw new NotFoundError('Organization')
    }

    return c.json({ data: null, message: 'Organization deleted successfully' })
  },
)
```

## Bad Endpoint Example (what not to do)

```ts
import { Hono } from 'hono'

const organizationsRoute = new Hono()

organizationsRoute.get('/', async (c) => {
  const db = c.get('db')
  const rows = await db.select().from(organizations)
  return c.json(rows)
})
```

Why bad:

- No auth middleware.
- No validation.
- DB logic in route handler.
- No DTO mapping.
- No stable response envelope.
