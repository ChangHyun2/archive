# Concurrent Optimistic Updates in React Query - Best Practices

## Best Practices

### 1. Always Cancel Queries Before Optimistic Updates

Cancel in-flight queries to avoid the "window of inconsistency" where optimistic updates get overwritten:

```typescript
const useToggleIsActive = (id: number) =>
  useMutation({
    mutationFn: api.toggleIsActive,
    onMutate: async () => {
      // ✅ Cancel in-flight queries to prevent overwriting optimistic update
      await queryClient.cancelQueries({
        queryKey: ['items', 'detail', id],
      })

      queryClient.setQueryData(['items', 'detail', id], (prevItem) =>
        prevItem
          ? {
              ...prevItem,
              isActive: !prevItem.isActive,
            }
          : undefined
      )
    },
    onSettled: () => {
      queryClient.invalidateQueries({
        queryKey: ['items', 'detail', id],
      })
    },
  })
```

**Why**: Without cancellation, if a mutation starts while a query is in-flight (e.g., from `refetchOnWindowFocus`), the optimistic update gets overwritten when that query finishes, causing UI to toggle back and forth.

### 2. Prevent Over-Invalidation with isMutating Check

Skip invalidations when related mutations are still running to avoid "window of inconsistency":

```typescript
onSettled: () => {
  // ✅ Only invalidate if no other related mutations are running
  if (queryClient.isMutating() === 1) {
    queryClient.invalidateQueries({
      queryKey: ['items', 'detail', id],
    })
  }
}
```

**Why**: 
- When `onSettled` is called, our own mutation is still in progress, so count is never 0
- Check for `=== 1` means "only our mutation is running"
- If another mutation is running, skip invalidation (it will invalidate when it finishes)

**Important**: Use `queryClient.isMutating()` (imperative), not `useIsMutating()` (hook), to avoid stale closure issues.

### 3. Limit Scope with mutationKey

For fine-grained invalidations, tag related mutations and check only those:

```typescript
const useToggleIsActive = (id: number) =>
  useMutation({
    mutationKey: ['items'],  // ✅ Tag related mutations
    mutationFn: api.toggleIsActive,
    onMutate: async () => {
      await queryClient.cancelQueries({
        queryKey: ['items', 'detail', id],
      })

      queryClient.setQueryData(['items', 'detail', id], (prevItem) =>
        prevItem
          ? {
              ...prevItem,
              isActive: !prevItem.isActive,
            }
          : undefined
      )
    },
    onSettled: () => {
      // ✅ Only check mutations with same mutationKey
      if (queryClient.isMutating({ mutationKey: ['items'] }) === 1) {
        queryClient.invalidateQueries({
          queryKey: ['items', 'detail', id],
        })
      }
    },
  })
```

**Why**: If you invalidate everything at the end anyway, wide scope is fine. But for fine-grained invalidations, limit the check to related mutations only.

### 4. Handle Complex Server Logic

Optimistic updates require **recreating server logic on the client**. This gets complex fast:

**Simple case** (toggle boolean):
```typescript
queryClient.setQueryData(['items', 'detail', item.id], (prevItem) =>
  prevItem
    ? {
        ...prevItem,
        isActive: !prevItem.isActive,
      }
    : undefined
)
```

**Complex case** (filtered list with multiple filters):
```typescript
// ⚠️ Must handle all filter logic
queryClient.setQueryData(['items', 'list', filters], (prevItems) =>
  prevItems
    ?.map((item) =>
      item.id === newItem.id ? { ...item, ...newItem } : item
    )
    .filter((item) => filters.categories.includes(item.category))
    // ... and text filters, and other filters
)
```

**Tradeoff**: Sometimes optimistic updates aren't worth the complexity. Consider if the UX improvement justifies duplicating server logic.

## Not To Do

### 1. Don't Skip Query Cancellation

Without `cancelQueries`, in-flight queries can overwrite your optimistic updates, causing flickering UI.

### 2. Don't Always Invalidate on Settled

If multiple mutations can run concurrently, always invalidating causes "window of inconsistency" when:
- Second mutation starts while first is running
- First mutation settles and invalidates
- Refetch from first invalidation finishes before second mutation
- UI reverts to old state

**Solution**: Check `isMutating()` before invalidating.

### 3. Don't Use useIsMutating Hook for This Check

Use `queryClient.isMutating()` (imperative) instead of `useIsMutating()` (hook) to avoid stale closure issues. The check needs to happen imperatively right before calling invalidation.

### 4. Don't Over-Complicate Optimistic Updates

If recreating server logic is too complex (multiple filters, sorting, etc.), consider:
- Not using optimistic updates for that case
- Using simpler invalidation instead
- Only doing optimistic updates for simple cases (toggles, simple adds)

## Key Insights

1. **Query cancellation is essential**: Prevents in-flight queries from overwriting optimistic updates
2. **Prevent over-invalidation**: Check `isMutating()` before invalidating to avoid windows of inconsistency
3. **Use mutationKey for scope**: Limit checks to related mutations when doing fine-grained invalidations
4. **Complexity tradeoff**: Optimistic updates require duplicating server logic - evaluate if it's worth it
5. **Concurrent mutations are tricky**: Edge cases can cause UI flickering - use cancellation + smart invalidation
