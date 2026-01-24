---
description: "React Query infinite queries: understand retry behavior applies to all pages, use maxPages to limit cache size, don't use same key as useQuery"
alwaysApply: false
---

# React Query Infinite Queries Rules

## Understand Retry Behavior

- **`retry: 3` means three retries over all pages, not three retries per page**
  - This is consistent with single queries - it's three retries per query, no matter how often it actually fetches
  - If page 3 fails during a refetch, the retry will restart from page 1

## Use maxPages to Limit Cache Size

- **If you have many pages in cache, use `maxPages` to limit cache size**

```typescript
useInfiniteQuery({
  queryKey: ['items'],
  queryFn: fetchPage,
  maxPages: 10,  // ✅ Only keep last 10 pages in cache
})
```

- **Why**: 
  - Prevents too many pages in cache
  - Speeds up rendering when navigating
  - Reduces memory usage

## Understand How Pages Are Structured

- **Infinite queries structure data as a linked list:**
  - Each page depends on the previous one
  - Pages are stored in `{ pages: [...] }` format
  - `getNextPageParam` determines the next page parameter

## Don't Do

- **Don't expect per-page retries**
  - Retries apply to the entire query operation, not individual pages
  - If page 3 fails during a refetch, the retry will restart from page 1

- **Don't use same `queryKey` for `useQuery` and `useInfiniteQuery`**

```typescript
// ❌ Wrong - they share the same cache entry
useQuery({
  queryKey: ['items'],
  queryFn: fetchItems,
})

useInfiniteQuery({
  queryKey: ['items'],  // ❌ Same key!
  queryFn: fetchPage,
})

// ✅ Correct - use different keys
useInfiniteQuery({
  queryKey: ['infiniteItems'],  // ✅ Different key
  queryFn: fetchPage,
})
```

- **Why**: They have fundamentally different data structures and would conflict.
