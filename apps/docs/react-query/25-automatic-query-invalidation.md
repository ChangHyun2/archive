# Automatic Query Invalidation after Mutations - Best Practices

## Best Practices

### 1. Use Global MutationCache Callbacks for Automatic Invalidation

React Query doesn't link Mutations to Queries by default because there's no one-size-fits-all solution. However, you can implement automatic invalidation using global cache callbacks:

```typescript
import { QueryClient, MutationCache } from '@tanstack/react-query'

const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: () => {
      queryClient.invalidateQueries()  // Invalidate everything
    },
  }),
})
```

**Why this works**: With just 5 lines, you get behavior similar to Remix/React Router - invalidate everything after every mutation.

**Important**: Invalidation doesn't always mean refetch. It:
- Refetches all **active** Queries (currently on screen)
- Marks the rest as **stale** (refetched when used next time)

This is usually a good trade-off - you get up-to-date data for what you're viewing without unnecessary requests.

### 2. Invalidate Everything by Default (Simple Approach)

**Best Practice**: Prefer invalidating everything over fine-grained revalidation.

**Why**:
- Code is as simple as it gets
- Better to fetch some data more often than strictly necessary than to miss a refetch
- Fine-grained revalidation is error-prone - you might forget to add a resource later
- With medium-sized `staleTime` (~2 minutes), impact is negligible

**Trade-off**: You might invalidate profile data when adding an issue, but this is usually acceptable.

### 3. Tie Invalidation to mutationKey

Use `mutationKey` to specify which Queries should be invalidated:

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
  mutationKey: ['issues'],  // Only invalidates issue-related queries
})
```

**Benefit**: Mutations without a key still invalidate everything (backward compatible).

### 4. Use Meta Option for Tagged Invalidation

Use `meta` to store tags for fine-grained invalidation:

```typescript
import { matchQuery } from '@tanstack/react-query'

const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: (_data, _variables, _context, mutation) => {
      queryClient.invalidateQueries({
        predicate: (query) =>
          // Invalidate all matching tags at once
          // or everything if no meta is provided
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

**TypeScript support**:
```typescript
declare module '@tanstack/react-query' {
  interface Register {
    mutationMeta: {
      invalidates?: Array<QueryKey>
    }
  }
}
```

### 5. Exclude Static Queries from Invalidation

Mark queries as "static" with `staleTime: Infinity` or `staleTime: 'static'` and exclude them from invalidation:

```typescript
const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: (_data, _variables, _context, mutation) => {
      const nonStaticQueries = (query: Query) => {
        const defaultStaleTime =
          queryClient.getQueryDefaults(query.queryKey).staleTime ?? 0
        const staleTimes = query.observers
          .map((observer) => observer.options.staleTime)
          .filter((staleTime) => staleTime !== undefined)

        const staleTime =
          query.getObserversCount() > 0
            ? Math.min(...staleTimes)
            : defaultStaleTime

        return staleTime !== Number.POSITIVE_INFINITY
      }

      queryClient.invalidateQueries({
        queryKey: mutation.options.mutationKey,
        predicate: nonStaticQueries,
      })
    },
  }),
})
```

**Update**: React Query now supports `staleTime: 'static'` to never trigger a refetch, even if manually invalidated.

### 6. Await Specific Invalidations When Needed

If you want the mutation to stay pending until specific queries refetch:

```typescript
const queryClient = new QueryClient({
  mutationCache: new MutationCache({
    onSuccess: () => {
      queryClient.invalidateQueries()  // Fire and forget
    },
  }),
})

useMutation({
  mutationFn: updateLabel,
  onSuccess: () => {
    // ✅ Returning the Promise to await it
    return queryClient.invalidateQueries(
      { queryKey: ['labels'] },
      { cancelRefetch: false }  // Important!
    )
  },
})
```

**How it works**:
1. Global callback invalidates everything (fire-and-forget)
2. Local callback awaits specific invalidation
3. Mutation stays pending until that refetch completes

**Note**: Use `cancelRefetch: false` to pick up the already in-flight Promise instead of creating a new request.

## Not To Do

### 1. Don't Over-Engineer Fine-Grained Invalidation

Fine-grained invalidation is nice if you know exactly what to refetch, but:
- It's error-prone (easy to forget resources)
- Requires maintenance when adding new resources
- Simple "invalidate everything" often works well enough

**Exception**: Use fine-grained invalidation only when you have a clear, stable schema of what affects what.

### 2. Don't Await All Invalidations

Awaiting all invalidations makes mutations slower. Only await what's critical for the user experience (e.g., the data they're currently viewing).

### 3. Don't Forget cancelRefetch When Awaiting

When awaiting a specific invalidation after global invalidation, use `cancelRefetch: false` to avoid duplicate requests.

## Key Insights

1. **Invalidation ≠ Refetch**: Only active queries refetch immediately; others are marked stale
2. **Simple is better**: Invalidate everything by default, add complexity only when needed
3. **Global callbacks**: Use MutationCache callbacks for automatic behavior
4. **Meta for flexibility**: Use `meta` option for tagged invalidation when needed
5. **Static queries**: Exclude queries with `staleTime: 'static'` from invalidation
