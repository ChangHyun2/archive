# Placeholder and Initial Data in React Query - Best Practices

## Key Concepts

### Cache Level vs Observer Level

- **Cache Level**: One cache entry per Query Key (shared globally)
  - Options: `queryFn`, `gcTime`
  - **initialData** works here

- **Observer Level**: Multiple subscriptions to the same cache entry
  - Options: `select`, `refetchInterval`
  - **placeholderData** works here

## Best Practices

### 1. Use initialData When Pre-filling from Another Query

**Best for**: Pre-filling a query from data in another query's cache.

```typescript
const useTodo = (id) => {
  const queryClient = useQueryClient()

  return useQuery({
    queryKey: ['todo', id],
    queryFn: () => fetchTodo(id),
    staleTime: 30 * 1000,
    initialData: () =>
      queryClient
        .getQueryData(['todo', 'list'])
        ?.find((todo) => todo.id === id),
    initialDataUpdatedAt: () =>
      // ✅ Will refetch in background if list data is older than staleTime
      queryClient.getQueryState(['todo', 'list'])?.dataUpdatedAt,
  })
}
```

**Why**: `initialData` is persisted to cache and respects `staleTime`. If data is fresh, no background refetch happens.

### 2. Use placeholderData for Everything Else

**Best for**: Showing "fake" data while real data is being fetched (e.g., skeleton loaders, default values).

```typescript
function Component() {
  const { data, status, isPlaceholderData } = useQuery({
    queryKey: ['number'],
    queryFn: fetchNumber,
    placeholderData: 23,  // "Fake" data
  })

  // ✅ Status is success immediately (no loading state)
  // ✅ Use isPlaceholderData to show visual hint
  return (
    <div>
      {isPlaceholderData && <span>Loading real data...</span>}
      <div>{data}</div>
    </div>
  )
}
```

**Why**: `placeholderData` is never persisted to cache and always triggers a background refetch.

### 3. Use Functions for Expensive Computations

Both `initialData` and `placeholderData` can be functions:

```typescript
// ✅ Function form for expensive computations
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  initialData: () => computeExpensiveInitialData(),
  placeholderData: () => generatePlaceholderData(),
})
```

### 4. Use initialDataUpdatedAt for Stale Data

When using `initialData` from another query, provide `initialDataUpdatedAt` to respect staleTime:

```typescript
useQuery({
  queryKey: ['todo', id],
  queryFn: () => fetchTodo(id),
  staleTime: 30 * 1000,
  initialData: () =>
    queryClient.getQueryData(['todo', 'list'])?.find((todo) => todo.id === id),
  initialDataUpdatedAt: () =>
    queryClient.getQueryState(['todo', 'list'])?.dataUpdatedAt,
})
```

**Why**: This tells React Query when the initialData was created, so it can decide whether to refetch based on `staleTime`.

## Differences

| Feature | initialData | placeholderData |
|---------|-------------|-----------------|
| **Level** | Cache level | Observer level |
| **Persistence** | ✅ Persisted to cache | ❌ Never persisted |
| **Background refetch** | Respects `staleTime` | Always refetches |
| **Multiple values** | Only first one used | Can be different per observer |
| **Error handling** | Keeps initialData on error | Shows error (no placeholder) |
| **Use case** | "Good" data from another query | "Fake" data while loading |

## When to Use What

### Use initialData when:
- Pre-filling from another query's cache
- You have "good" data that's as valid as fetched data
- You want to respect `staleTime` and avoid unnecessary refetches

### Use placeholderData when:
- Showing skeleton loaders or default values
- You want to always fetch real data in the background
- Different components might need different placeholder values
- You want to show visual hints with `isPlaceholderData`

## Not To Do

### 1. Don't Use initialData for "Fake" Data

`initialData` is treated as "real" data. If you use it for placeholder values, React Query won't refetch when it should.

### 2. Don't Forget initialDataUpdatedAt

When using `initialData` from another query, always provide `initialDataUpdatedAt` to respect `staleTime` correctly.

### 3. Don't Expect Multiple initialData Values

Only the first observer's `initialData` is used (cache level). Subsequent observers with different `initialData` won't affect the cache.

### 4. Don't Use placeholderData for Pre-filling from Cache

Use `initialData` instead - it persists to cache and respects `staleTime`.

## Key Insights

1. **Both avoid loading states**: Query goes directly to success state
2. **Neither works if cache has data**: If data exists in cache, both are ignored
3. **initialData = "real" data**: Treated as valid, persisted, respects staleTime
4. **placeholderData = "fake" data**: Never persisted, always refetches, use `isPlaceholderData` flag
