---
name: react-table-factory-patterns
description: Patterns for using a `createTablePropsFactory` + `columnBuilders` abstraction in React route components — paginated lists, expandable rows, highlight state, and the wrapper-vs-manual-hook decision. Use when adding a new paginated table to a route, deciding whether to reach for the factory or write a manual hook, building columns with the column-builder DSL, or reviewing a route component's table setup. Specific to projects that ship the `createTablePropsFactory(...)` and `buildColumns/columnBuilders` API; not transferable to projects without that abstraction.
---

# React Table Factory Patterns

Use this skill when working in a codebase that exposes a `createTablePropsFactory` hook factory together with `buildColumns` / `columnBuilders` for column definitions. The factory exists to remove the repetitive plumbing of pagination + page params + data fetching for standard list tables; this skill encodes when to use it and when to drop down to a manual hook.

## Use this skill when

- Adding a new paginated table to a route component
- Deciding whether to use the factory or write a manual table hook
- Building columns with the column-builder DSL
- Adding highlight state, expandable rows, or row-level mutations to a table
- Reviewing a route's table wiring during code review

## When to use the factory vs a manual hook

| Scenario                            | Use Factory  | Manual Hook |
| ----------------------------------- | ------------ | ----------- |
| Standard paginated table            | ✅           |             |
| Custom local state (view more/less) | ✅ wrapper   |             |
| Highlight state auto-reset          | ✅ wrapper   |             |
| Nested expandable tables            | ✅           |             |
| Client-side pagination              |              | ✅          |
| Editable table with mutations       |              | ✅          |
| Local state for optimistic updates  |              | ✅          |
| Custom query enable conditions      |              | ✅          |

**Factory-incompatible patterns** (always reach for a manual hook): inline editing (`editable` columns), `setDataSource` for optimistic updates, custom API enable logic, or anything that needs to mutate the displayed rows in place without a refetch.

## Quick start — basic pattern

```ts
import { createTablePropsFactory, buildColumns, columnBuilders } from '~/hooks/table-factories'
import { useEntityList } from '../api/useEntityList'
import { usePageParams } from './usePageParams'

export const useTableProps = createTablePropsFactory<EntityDTO, EntityDTO, Params>({
  useDataFetcher: useEntityList,
  usePageParams,
})
```

The factory returns a hook that exposes pagination, search params, and `dataSource` ready for direct consumption.

## Wrapper pattern — adding custom state

When you need highlight state, expandable rows, or other per-table state that the factory doesn't model, wrap the factory hook rather than passing extra hooks through `useExtraHooks`:

```ts
const useFactoryProps = createTablePropsFactory<EntityDTO, EntityDTO, Params>({
  useDataFetcher: useEntityList,
  usePageParams,
})

export const useTableProps = () => {
  const tableProps = useFactoryProps()
  const { data: highlightState, resetData } = useTableHighlightState()

  // Auto-reset highlight on data refresh
  useEffect(() => {
    if (!tableProps.isDataSourceFetching && highlightState) {
      resetData()
    }
  }, [tableProps.isDataSourceFetching])

  return { ...tableProps, columns: useTableColumns }
}
```

## Column builders

Available builders in `columnBuilders`:

| Builder                             | Usage                                |
| ----------------------------------- | ------------------------------------ |
| `text(key, title, opts?)`           | Text column with `'-'` fallback      |
| `number(key, title, opts?)`         | Numeric column with `0` fallback     |
| `date(key, title, opts?)`           | Date formatting with custom format   |
| `timestamp()`                       | Created/Updated timestamps           |
| `image(key, title, opts?)`          | Image with placeholder fallback      |
| `link(key, title, toFn, opts?)`     | Link column with navigation          |
| `nested(path, title, opts?)`        | Nested field (e.g., `customer.name`) |
| `status(key, statusMap, opts?)`     | Status tag with color mapping        |
| `actions(actions[], opts?)`         | Action buttons/links                 |
| `custom(key, title, render, opts?)` | Custom render escape hatch           |

```ts
const columns = buildColumns<EntityDTO>(
  columnBuilders.text('name', 'Name', { width: 200 }),
  columnBuilders.number('quantity', 'Qty', { sorter: true }),
  columnBuilders.timestamp(),
  columnBuilders.actions([
    { label: 'Edit', onClick: onEdit },
    { label: 'Delete', onClick: onDelete },
  ])
)
```

## Factory options reference

| Option           | Purpose                                                                |
| ---------------- | ---------------------------------------------------------------------- |
| `useDataFetcher` | Hook that fetches paginated data                                       |
| `usePageParams`  | Hook returning `{ pageParams, searchParams }`                          |
| `dataTransform`  | Transform raw items (e.g., flatten variants)                           |
| `extraReturns`   | Add extra values to return (e.g., response metadata)                   |
| `rowKey`         | Custom row key (default: `'id'`)                                       |
| `rowClassName`   | Row class generator with extraHooks access                             |
| `expandable`     | Expandable row configuration                                           |
| `getDataSource`  | Custom data extractor (default: `response.data.items`)                 |
| `getTotal`       | Custom total extractor (default: `response.data.meta.itemCount`)       |
| `useExtraHooks`  | Additional hooks (highlight state, etc.) — prefer wrapper pattern      |
| `onDataFetched`  | Callback after data fetch completes                                    |

## Workflow: adding a new table to a route

1. Decide factory vs manual hook (see decision matrix above). If anything in the "Manual Hook" column applies, stop here and write a manual hook.
2. Create `usePageParams.ts` returning `{ pageParams, searchParams }` — this is the contract the factory expects.
3. Create the API list hook (e.g., `useEntityList`) that accepts `params` and returns the paginated response.
4. Build the factory hook in the route's `hooks/` directory: `useTableProps = createTablePropsFactory(...)`.
5. If the table needs highlight/expandable/local state, switch to the **wrapper pattern** before adding `useExtraHooks` — wrappers compose more cleanly.
6. Define columns via `buildColumns(...)` + `columnBuilders.*`. Reach for `columnBuilders.custom` only when none of the standard builders fit.
7. Wire into the route: `const { dataSource, columns: tableColumns, ...rest } = useTableProps()`. Don't alias `dataSource` unless there's a real naming conflict; alias `columns` only because it's a function that needs to be called with action handlers.

## Common pitfalls

- **Reaching for `useExtraHooks` instead of a wrapper** — wrappers are easier to test, easier to read, and don't bury the table's state behind an extra layer of indirection. Use `useExtraHooks` only when the wrapper pattern can't access the data flow you need.
- **Aliasing `dataSource` unnecessarily** — `const { dataSource: customerList } = useTableProps()` is noise unless there's a name clash. The property name describes what it is.
- **Forcing the factory onto editable/optimistic tables** — they need a manual hook. The factory's contract assumes the data comes from a refetchable query.
- **Putting business logic in column builders** — keep column builders declarative. Put computed columns in `dataTransform` or in the underlying fetcher.

## When NOT to use this skill

If you're in a project that doesn't ship a `createTablePropsFactory` abstraction, this skill doesn't apply. The general lessons (factory-vs-manual decision matrix, declarative columns, wrapper-over-injection for custom state) translate, but the API names won't.
