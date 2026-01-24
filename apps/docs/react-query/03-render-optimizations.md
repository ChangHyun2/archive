# React Query Render Optimizations - Best Practices

## Important Disclaimer

**Render optimizations are advanced** - React Query already has excellent defaults. Most apps don't need further optimizations.

**Key principle**: "Unnecessary re-renders" are better than "missing renders that should have been there". Re-renders ensure your app stays up-to-date.

## Best Practices

### 1. Use Tracked Queries (v4+ Default)

**Best Practice**: Use `notifyOnChangeProps: 'tracked'` to automatically track which fields you use:

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      notifyOnChangeProps: 'tracked',  // ✅ Automatic tracking
    },
  },
})
```

**How it works**: React Query tracks which fields you access during render and only notifies when those specific fields change.

**Benefits**:
- No manual list maintenance
- Automatically stays in sync with what you actually use
- Same optimization as manual list, but automatic

**Update**: Starting with v4, tracked queries are **turned on by default**. You can opt out with `notifyOnChangeProps: 'all'`.

### 2. Understand isFetching Re-renders

Even with `select`, components will re-render twice during background refetches:

```typescript
// Component re-renders twice during background refetch:
// 1. { status: 'success', data: 2, isFetching: true }
// 2. { status: 'success', data: 2, isFetching: false }
```

**Why**: `isFetching` changes during refetch, even if `data` doesn't.

**Solution**: Use `notifyOnChangeProps: ['data']` to only notify on data changes (or use tracked queries).

### 3. Manual notifyOnChangeProps (When Needed)

If you need fine-grained control, specify which props to track:

```typescript
export const useTodosQuery = (select, notifyOnChangeProps) =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select,
    notifyOnChangeProps,  // e.g., ['data', 'error']
  })

export const useTodosCount = () =>
  useTodosQuery((data) => data.length, ['data'])
```

**Warning**: Keep the list in sync with what you actually use, or components won't update when needed fields change.

### 4. Understand Structural Sharing

React Query uses **structural sharing** by default - it keeps referential identity of unchanged data:

```typescript
// Old data
[
  { id: 1, name: "Learn React", status: "active" },
  { id: 2, name: "Learn React Query", status: "todo" }
]

// New data after refetch
[
  { id: 1, name: "Learn React", status: "done" },  // ✅ New reference
  { id: 2, name: "Learn React Query", status: "todo" }  // ✅ Same reference!
]
```

**Benefits**:
- Unchanged objects keep same reference
- Works great with selectors for partial subscriptions
- Prevents unnecessary re-renders when data hasn't actually changed

**Note**: For selectors, structural sharing happens twice:
1. On result from `queryFn` (to detect changes)
2. On result from `select` function

**Disable if needed**: Set `structuralSharing: false` if it becomes a bottleneck with very large datasets.

## Not To Do

### 1. Don't Use Object Rest Destructuring with Tracked Queries

```typescript
// 🚨 Will track all fields (defeats the purpose)
const { isLoading, ...queryInfo } = useQuery(...)

// ✅ This is fine - normal destructuring
const { isLoading, data } = useQuery(...)
```

**Why**: Rest destructuring accesses all fields, so all fields get tracked.

### 2. Don't Access Fields Only in Effects

Tracked queries only work "during render". Fields accessed only in effects won't be tracked:

```typescript
const queryInfo = useQuery(...)

// 🚨 Will not correctly track data
React.useEffect(() => {
  console.log(queryInfo.data)
})

// ✅ Fine - dependency array is accessed during render
React.useEffect(() => {
  console.log(queryInfo.data)
}, [queryInfo.data])
```

### 3. Don't Hard-code notifyOnChangeProps in Custom Hooks

```typescript
// 🚨 Hook doesn't know what component will use
export const useTodosCount = () =>
  useTodosQuery((data) => data.length, ['data'])

function TodosCount() {
  // Component uses error, but won't get notified!
  const { error, data } = useTodosCount()
  return <div>{error ? error : null}</div>
}
```

**Solution**: Use tracked queries (automatic) or pass `notifyOnChangeProps` from component.

### 4. Don't Worry About Conditional Field Access

Tracked queries don't reset on each render. If you track a field once, it's tracked for the observer's lifetime:

```typescript
const queryInfo = useQuery(...)

if (someCondition()) {
  // 🟡 Will track data field if condition was true in ANY previous render
  return <div>{queryInfo.data}</div>
}
```

This is usually fine, but be aware of the behavior.

### 5. Don't Prematurely Optimize

Most apps don't need render optimizations. React Query's defaults are excellent. Only optimize when you have a proven performance problem.

## Key Insights

1. **Tracked queries are default (v4+)**: Automatically track what you use
2. **isFetching causes re-renders**: Even if data doesn't change, `isFetching` changes during refetch
3. **Structural sharing is automatic**: Keeps referential identity for unchanged data
4. **Re-renders are usually fine**: Better than missing updates
5. **Don't over-optimize**: React Query's defaults work well for most apps
