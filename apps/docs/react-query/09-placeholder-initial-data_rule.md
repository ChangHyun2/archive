---
description: "React Query placeholder vs initial data: use initialData for pre-filling from cache, placeholderData for fake data, always provide initialDataUpdatedAt"
alwaysApply: false
---

# React Query Placeholder and Initial Data Rules

## Use initialData When Pre-filling from Another Query

- **Use `initialData` for pre-filling a query from data in another query's cache**

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

- **Why**: `initialData` is persisted to cache and respects `staleTime`. If data is fresh, no background refetch happens.

## Use placeholderData for Everything Else

- **Use `placeholderData` for showing "fake" data while real data is being fetched**

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

- **Why**: `placeholderData` is never persisted to cache and always triggers a background refetch.

## Use initialDataUpdatedAt for Stale Data

- **When using `initialData` from another query, always provide `initialDataUpdatedAt`**

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

- **Why**: This tells React Query when the initialData was created, so it can decide whether to refetch based on `staleTime`.

## Differences

| Feature | initialData | placeholderData |
|---------|-------------|-----------------|
| **Level** | Cache level | Observer level |
| **Persistence** | ✅ Persisted to cache | ❌ Never persisted |
| **Background refetch** | Respects `staleTime` | Always refetches |
| **Use case** | "Good" data from another query | "Fake" data while loading |

## Don't Do

- **Don't use `initialData` for "fake" data**
  - `initialData` is treated as "real" data
  - If you use it for placeholder values, React Query won't refetch when it should

- **Don't forget `initialDataUpdatedAt`**
  - When using `initialData` from another query, always provide `initialDataUpdatedAt` to respect `staleTime` correctly

- **Don't expect multiple `initialData` values**
  - Only the first observer's `initialData` is used (cache level)
  - Subsequent observers with different `initialData` won't affect the cache

- **Don't use `placeholderData` for pre-filling from cache**
  - Use `initialData` instead - it persists to cache and respects `staleTime`
