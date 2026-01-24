# React Query Error Handling - Best Practices

## Best Practices

### 1. Use Error Boundaries for Rendering Errors

Error Boundaries catch runtime errors during rendering. React Query can propagate errors to Error Boundaries by using `throwOnError`:

```typescript
function TodoList() {
  // ✅ Will propagate all fetching errors to the nearest Error Boundary
  const todos = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    throwOnError: true,
  })

  if (todos.data) {
    return (
      <div>
        {todos.data.map((todo) => (
          <Todo key={todo.id} {...todo} />
        ))}
      </div>
    )
  }

  return 'Loading...'
}
```

**Granular error boundaries** (v3.23.0+): Customize which errors go to Error Boundary:

```typescript
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  // ✅ Only server errors (5xx) will go to the Error Boundary
  // 4xx errors can be handled locally
  throwOnError: (error) => error.response?.status >= 500,
})
```

**Why**: This works great for form submissions - 4xx validation errors handled locally, 5xx server errors go to Error Boundary.

**Update**: Before v5, `throwOnError` was known as `useErrorBoundary`.

### 2. Use Global Callbacks for Error Notifications

**Don't use `onError` on individual queries** if you want to show error toasts - it will fire for every Observer (every component using the hook).

```typescript
// ⚠️ Looks good, but will show multiple toasts if hook is used multiple times
const useTodos = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    onError: (error) =>
      toast.error(`Something went wrong: ${error.message}`),
  })
```

**Use global QueryCache callbacks instead**:

```typescript
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) =>
      toast.error(`Something went wrong: ${error.message}`),
  }),
})
```

This will show an error toast **once per request**, which is exactly what we want.

### 3. Show Error Toasts Only for Background Refetches

Keep stale UI intact and only notify about background update failures:

```typescript
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      // ✅ Only show error toasts if we already have data in the cache
      // which indicates a failed background update
      if (query.state.data !== undefined) {
        toast.error(`Something went wrong: ${error.message}`)
      }
    },
  }),
})
```

**Why**: If we already have data, this is a background refetch failure. We want to keep showing the stale data but notify the user. For initial fetches, handle errors locally or with Error Boundaries.

### 4. Use Global Callbacks for Error Tracking

The global QueryCache `onError` callback is the **best place for error tracking/monitoring** because:
- Guaranteed to run only once per request
- Cannot be overwritten like `defaultOptions`
- Centralized error handling

### 5. Three Main Error Handling Approaches

You can mix and match:

1. **The `error` property** from `useQuery` - handle locally in component
2. **The `onError` callback** - on query itself or global QueryCache/MutationCache
3. **Error Boundaries** - using `throwOnError`

**Recommended pattern**: Show error toasts for background refetches (keep stale UI intact) and handle everything else locally or with Error Boundaries.

## Not To Do

### 1. Don't Use `onError` on Custom Hooks for Toasts

If you use `onError` in a custom hook, it will fire for **every Observer** (every component using that hook). This means multiple toasts for a single failed request.

```typescript
// 🚨 Will show multiple toasts if useTodos is used in multiple components
const useTodos = () => {
  const todos = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos
  })

  React.useEffect(() => {
    if (todos.error) {
      toast.error(`Something went wrong: ${todos.error.message}`)
    }
  }, [todos.error])

  return todos
}
```

**Solution**: Use global QueryCache callbacks instead.

### 2. Don't Unmount UI on Background Errors

Don't unmount your complete UI just because a background refetch failed. The API might be temporarily down or you might have hit a rate limit.

```typescript
// ❌ Bad: Unmounts everything on background error
if (todos.isError) {
  return 'An error occurred'
}
```

**Better**: Show error toasts for background refetches while keeping stale UI visible.

### 3. Don't Forget to Transform Errors for fetch API

React Query needs a **rejected Promise** to handle errors correctly. If you use `fetch` API or other libraries that don't reject on 4xx/5xx status codes, transform them yourself:

```typescript
const queryFn = async () => {
  const response = await fetch('/api/todos')
  if (!response.ok) {
    throw new Error('Failed to fetch')  // ✅ Transform to rejected Promise
  }
  return response.json()
}
```

## Prerequisites

- React Query needs a **rejected Promise** for error handling
- Libraries like `axios` provide this automatically
- `fetch` API and some other libraries require manual transformation
