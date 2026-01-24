---
description: "Why React Query: fixes race conditions, provides loading state, distinguishes empty vs no data, resets state on parameter change, cancels requests"
alwaysApply: false
---

# Why You Want React Query Rules

## React Query Fixes Common useEffect Bugs

### Fix Race Conditions

- **React Query stores state by `queryKey` (input), preventing race conditions**

```typescript
// ✅ No race condition - state stored by queryKey
function Bookmarks({ category }) {
  const { isLoading, data, error } = useQuery({
    queryKey: ['bookmarks', category],  // ✅ Different key = different cache entry
    queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
  })
}
```

- **Why**: Each `queryKey` is a separate cache entry. Changing category = different key = separate state.

### Provide Loading State

- **React Query provides loading state automatically**

```typescript
// ✅ Loading state included
const { isLoading, data, error } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})

if (isLoading) return <SkeletonLoader />
```

### Distinguish Empty vs No Data

- **React Query separates these states**

```typescript
// ✅ Clear separation
const { isLoading, data, error } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})

// isLoading = true → no data yet
// isLoading = false && data = [] → no data at all
// isLoading = false && data = [...] → has data
```

### Reset State on Parameter Change

- **React Query automatically resets state per `queryKey`**

```typescript
// ✅ Each queryKey has its own state
const { data, error } = useQuery({
  queryKey: ['bookmarks', category],  // ✅ Different category = clean state
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})
```

- **Why**: Each `queryKey` is independent. Changing category = new query = fresh state.

### Cancel Requests

- **React Query automatically cancels requests when component unmounts or query key changes**

```typescript
// ✅ Automatic cancellation
useQuery({
  queryKey: ['bookmarks', category],
  queryFn: ({ signal }) => fetch(`${endpoint}/${category}`, { signal }).then(res => res.json()),
})
```

- **Why**: Prevents memory leaks and race conditions from stale responses.

## Common Bugs in useEffect Data Fetching

The "simple" useEffect example has 5 bugs:
1. Race conditions (responses arrive out of order)
2. No loading state
3. Can't distinguish empty vs no data
4. State doesn't reset on parameter change
5. No request cancellation

React Query fixes all of these automatically.
