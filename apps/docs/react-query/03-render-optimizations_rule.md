---
description: "React Query render optimizations: tracked queries, notifyOnChangeProps, structural sharing - use defaults unless proven performance issue"
alwaysApply: false
---

# React Query Render Optimizations Rules

## Use Tracked Queries (v4+ Default)

- **Tracked queries are enabled by default in v4+**
  - Automatically tracks which fields you use during render
  - Only notifies when those specific fields change
  - No manual list maintenance needed

```typescript
// ✅ Default behavior (v4+)
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      notifyOnChangeProps: 'tracked',  // ✅ Default in v4+
    },
  },
})
```

## Understand isFetching Re-renders

- **Components will re-render twice during background refetches** even with `select`
  - First: `{ status: 'success', data: 2, isFetching: true }`
  - Second: `{ status: 'success', data: 2, isFetching: false }`
- **Solution**: Use `notifyOnChangeProps: ['data']` to only notify on data changes (or use tracked queries)

## Structural Sharing

- **React Query uses structural sharing by default**
  - Keeps referential identity of unchanged data
  - Unchanged objects keep same reference
  - Works great with selectors for partial subscriptions
- **Disable if needed**: Set `structuralSharing: false` if it becomes a bottleneck with very large datasets

## Don't Do

- **Don't use object rest destructuring with tracked queries**
  - Rest destructuring accesses all fields, so all fields get tracked

```typescript
// ❌ Wrong - will track all fields
const { isLoading, ...queryInfo } = useQuery(...)

// ✅ Correct - normal destructuring
const { isLoading, data } = useQuery(...)
```

- **Don't access fields only in effects**
  - Tracked queries only work "during render"
  - Fields accessed only in effects won't be tracked

```typescript
// ❌ Wrong - will not correctly track data
const queryInfo = useQuery(...)
React.useEffect(() => {
  console.log(queryInfo.data)
})

// ✅ Correct - dependency array is accessed during render
React.useEffect(() => {
  console.log(queryInfo.data)
}, [queryInfo.data])
```

- **Don't hard-code `notifyOnChangeProps` in custom hooks**
  - Hook doesn't know what component will use
  - Solution: Use tracked queries (automatic) or pass `notifyOnChangeProps` from component

- **Don't prematurely optimize**
  - Most apps don't need render optimizations
  - React Query's defaults are excellent
  - Only optimize when you have a proven performance problem
