# Status Checks in React Query - Best Practices

## Key Concepts

### Query States

React Query exposes status fields derived from an internal state machine. A query can be in one of these states:

- **`success`**: Query was successful, and you have `data`
- **`error`**: Query failed, and an `error` is set
- **`pending`**: Query has no data (was `loading` before v5, `idle` before v4)

**Important**: `isFetching` is **not** part of the state machine - it's an additional flag that's `true` whenever a request is in-flight. You can be `fetching` and `success`, or `fetching` and `error` - but you cannot be `loading` and `success` at the same time.

## Best Practices

### 1. Check Data Availability First (Data-First Pattern)

**Best Practice**: Check for data availability first, then error, then pending:

```typescript
const todos = useTodos()

// ✅ Check data first
if (todos.data) {
  return (
    <div>
      {todos.data.map(renderTodo)}
    </div>
  )
}

// Then check error
if (todos.error) {
  return 'An error has occurred: ' + todos.error.message
}

// Finally check pending
return 'Loading...'
```

**Why**: React Query uses stale-while-revalidate caching. When a background refetch fails, you'll have **both** error and stale data available. Showing stale data is usually better UX than immediately showing an error screen.

### 2. Understand Background Refetch Behavior

React Query refetches aggressively by default:
- `refetchOnMount`
- `refetchOnWindowFocus`
- `refetchOnReconnect`

**Scenarios where background refetch can fail**:
1. User opens page, works on it, switches tabs, comes back → background refetch fails
2. User navigates from list → detail → back to list → detail again → sees cached data, but background refetch fails

**In both cases**: Query state will be `{ status: 'error', error: {...}, data: [...] }` - both error and stale data exist.

### 3. Decide What to Show Based on Use Case

There's no one-size-fits-all answer. Consider:

- **Show stale data only**: If background errors aren't critical
- **Show both**: Stale data with a background error indicator
- **Show error only**: If data accuracy is critical (rare)

**Key principle**: Don't replace working data with an error screen just because a background refetch failed.

## Not To Do

### 1. Don't Check Status Before Data (Standard Pattern)

```typescript
// 🚨 Standard pattern - can be confusing UX
const todos = useTodos()

if (todos.isPending) {
  return 'Loading...'
}

if (todos.error) {
  return 'An error has occurred: ' + todos.error.message
}

return (
  <div>
    {todos.data.map(renderTodo)}
  </div>
)
```

**Why this is problematic**:
- If background refetch fails, you'll show error screen even though stale data exists
- React Query retries 3 times by default with exponential backoff
- User might see data disappear and error appear after a few seconds
- Very confusing UX, especially without background fetching indicator

### 2. Don't Ignore Stale Data When Error Exists

```typescript
// 🚨 Replaces stale data with error
if (todos.error) {
  return <ErrorScreen />
}

// This never runs if error exists, even if stale data is available
return <DataDisplay data={todos.data} />
```

**Why**: React Query's stale-while-revalidate means you can have both error and data. Don't throw away usable data.

### 3. Don't Assume Error Means No Data

```typescript
// 🚨 Wrong assumption
if (todos.error) {
  // Assumes no data exists
  return <EmptyState />
}
```

**Reality**: With stale-while-revalidate, you often have both error and data. Check data availability first.

## Key Insights

1. **Stale-while-revalidate**: React Query embraces this pattern - you can have both error and data
2. **Background refetches can fail**: Aggressive refetching means background errors are common
3. **Data-first pattern**: Check data availability before error/pending for better UX
4. **No one-size-fits-all**: Decision depends on your use case
5. **isFetching vs status**: `isFetching` is separate from state machine - can be true with success or error
