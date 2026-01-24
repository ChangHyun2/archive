---
description: "React Query automatic invalidation: use global MutationCache callbacks, invalidate everything, use mutationKey or meta option"
alwaysApply: false
---

# React Query Automatic Query Invalidation Rules

## Use Global MutationCache Callbacks

- **Implement automatic invalidation using global cache callbacks**

```typescript
import { QueryClient, MutationCache } from '@tanstack/react-query'

const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: () => {
      queryClient.invalidateQueries()  // ✅ Invalidate everything
    },
  }),
})
```

- **Why this works**: With just 5 lines, you get behavior similar to Remix/React Router - invalidate everything after every mutation.

- **Important**: Invalidation doesn't always mean refetch. It:
  - Refetches all **active** Queries (currently on screen)
  - Marks the rest as **stale** (refetched when used next time)

## Invalidate Everything by Default

- **Prefer invalidating everything over fine-grained revalidation**

**Why**:
- Code is as simple as it gets
- Better to fetch some data more often than strictly necessary than to miss a refetch
- Fine-grained revalidation is error-prone - you might forget to add a resource later
- With medium-sized `staleTime` (~2 minutes), impact is negligible

## Tie Invalidation to mutationKey

- **Use `mutationKey` to specify which Queries should be invalidated**

```typescript
const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: (_data, _variables, _context, mutation) => {
      queryClient.invalidateQueries({
        queryKey: mutation.options.mutationKey,
      })
    },
  }),
})

// Usage:
useMutation({
  mutationFn: updateIssue,
  mutationKey: ['issues'],  // ✅ Only invalidates issue-related queries
})
```

- **Benefit**: Mutations without a key still invalidate everything (backward compatible).

## Use Meta Option for Tagged Invalidation

- **Use `meta` to store tags for fine-grained invalidation**

```typescript
import { matchQuery } from '@tanstack/react-query'

const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: (_data, _variables, _context, mutation) => {
      queryClient.invalidateQueries({
        predicate: (query) =>
          mutation.meta?.invalidates?.some((queryKey) =>
            matchQuery({ queryKey }, query)
          ) ?? true,
      })
    },
  }),
})

// Usage:
useMutation({
  mutationFn: updateLabel,
  meta: {
    invalidates: [['issues'], ['labels']],
  },
})
```

## Use staleTime: 'static' for Better Control

- **v5+ provides `staleTime: 'static'` which prevents automatic refetches**

```typescript
useQuery({
  queryKey: ['config'],
  queryFn: fetchConfig,
  staleTime: 'static',  // ✅ Never goes stale automatically
})
```

- **Why**: Data only goes stale when explicitly invalidated, not on time-based triggers.

## Await Invalidations

- **Await invalidations to keep mutation in loading state**

```typescript
onSuccess: async () => {
  await queryClient.invalidateQueries({
    queryKey: ['items'],
  })
}
```

- **Why**: Keeps mutation in loading state while queries update, providing better UX.

## Don't Do

- **Don't implement fine-grained invalidation unless necessary**
  - Error-prone - you might forget to add a resource later
  - Fix: Prefer invalidating everything, use `mutationKey` or `meta` for targeted invalidation

- **Don't forget to await invalidations**
  - Mutation completes before queries update
  - Fix: Await `invalidateQueries` to keep mutation in loading state
