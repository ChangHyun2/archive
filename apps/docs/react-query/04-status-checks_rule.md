---
description: "React Query status checks: use data-first pattern, understand stale-while-revalidate, don't replace working data with error screen"
alwaysApply: false
---

# React Query Status Checks Rules

## Data-First Pattern

- **Check data availability first, then error, then pending**
  - React Query uses stale-while-revalidate caching
  - When a background refetch fails, you'll have both error and stale data available
  - Showing stale data is usually better UX than immediately showing an error screen

```typescript
const todos = useTodos()

// ✅ Check data first
if (todos.data) {
  return <div>{todos.data.map(renderTodo)}</div>
}

// Then check error
if (todos.error) {
  return 'An error has occurred: ' + todos.error.message
}

// Finally check pending
return 'Loading...'
```

## Understand Background Refetch Behavior

- **React Query refetches aggressively by default**
  - `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`
- **Background refetch can fail** while stale data exists
  - Query state will be `{ status: 'error', error: {...}, data: [...] }`
  - Both error and stale data exist simultaneously

## Don't Do

- **Don't check status before data (standard pattern)**
  - If background refetch fails, you'll show error screen even though stale data exists
  - Very confusing UX, especially without background fetching indicator

```typescript
// ❌ Wrong - can show error when stale data exists
if (todos.isPending) {
  return 'Loading...'
}

if (todos.error) {
  return 'An error has occurred'
}

return <div>{todos.data.map(renderTodo)}</div>
```

- **Don't ignore stale data when error exists**
  - React Query's stale-while-revalidate means you can have both error and data
  - Don't throw away usable data

```typescript
// ❌ Wrong - replaces stale data with error
if (todos.error) {
  return <ErrorScreen />
}

// This never runs if error exists, even if stale data is available
return <DataDisplay data={todos.data} />
```

- **Don't assume error means no data**
  - With stale-while-revalidate, you often have both error and data
  - Check data availability first
