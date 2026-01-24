---
description: "React Query concurrent optimistic updates: always cancel queries before updates, prevent over-invalidation with isMutating check, limit scope with mutationKey"
alwaysApply: false
---

# React Query Concurrent Optimistic Updates Rules

## Always Cancel Queries Before Optimistic Updates

- **Cancel in-flight queries to avoid the "window of inconsistency"**

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
  })
```

- **Why**: Without cancellation, if a mutation starts while a query is in-flight (e.g., from `refetchOnWindowFocus`), the optimistic update gets overwritten when that query finishes, causing UI to toggle back and forth.

## Prevent Over-Invalidation with isMutating Check

- **Skip invalidations when related mutations are still running**

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

- **Why**: 
  - When `onSettled` is called, our own mutation is still in progress, so count is never 0
  - Check for `=== 1` means "only our mutation is running"
  - If another mutation is running, skip invalidation (it will invalidate when it finishes)

- **Important**: Use `queryClient.isMutating()` (imperative), not `useIsMutating()` (hook), to avoid stale closure issues.

## Limit Scope with mutationKey

- **For fine-grained invalidations, tag related mutations and check only those**

```typescript
const useToggleIsActive = (id: number) =>
  useMutation({
    mutationKey: ['items'],  // ✅ Tag related mutations
    mutationFn: api.toggleIsActive,
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

- **Why**: If you invalidate everything at the end anyway, wide scope is fine. But for fine-grained invalidations, limit the check to related mutations only.

## Handle Complex Server Logic

- **Optimistic updates require recreating server logic on the client**
  - This gets complex fast
  - Simple case (toggle boolean): Easy to implement
  - Complex case (server calculates new state): Hard to implement correctly
  - Consider if optimistic updates are worth the complexity

## Don't Do

- **Don't skip query cancellation**
  - Optimistic updates get overwritten by in-flight queries
  - Fix: Always cancel queries before optimistic updates

- **Don't always invalidate on settled**
  - Causes over-invalidation when multiple mutations run concurrently
  - Fix: Check `isMutating` before invalidating

- **Don't use `useIsMutating` for checks**
  - Hook can have stale closure issues
  - Fix: Use `queryClient.isMutating()` (imperative)

- **Don't over-complicate optimistic updates**
  - Only use when instant feedback is truly required
  - Fix: Sometimes it's enough to disable the button and show a loading animation
