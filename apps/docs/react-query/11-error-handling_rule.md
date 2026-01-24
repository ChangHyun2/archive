---
description: "React Query error handling: use Error Boundaries with throwOnError, global callbacks for notifications, show toasts for background refetches only"
alwaysApply: false
---

# React Query Error Handling Rules

## Use Error Boundaries for Rendering Errors

- **Use `throwOnError` to propagate errors to Error Boundaries**

```typescript
function TodoList() {
  // ✅ Will propagate all fetching errors to the nearest Error Boundary
  const todos = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    throwOnError: true,
  })
}
```

- **Granular error boundaries (v3.23.0+)**: Customize which errors go to Error Boundary

```typescript
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  // ✅ Only server errors (5xx) will go to the Error Boundary
  // 4xx errors can be handled locally
  throwOnError: (error) => error.response?.status >= 500,
})
```

## Use Global Callbacks for Error Notifications

- **Don't use `onError` on individual queries** if you want to show error toasts
  - It will fire for every Observer (every component using the hook)
- **Use global QueryCache callbacks instead**

```typescript
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) =>
      toast.error(`Something went wrong: ${error.message}`),
  }),
})
```

- This will show an error toast **once per request**, which is exactly what we want.

## Show Error Toasts Only for Background Refetches

- **Keep stale UI intact and only notify about background update failures**

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

- **Why**: If we already have data, this is a background refetch failure. We want to keep showing the stale data but notify the user. For initial fetches, handle errors locally or with Error Boundaries.

## Transform Errors for fetch API

- **React Query needs a rejected Promise for error handling**
  - If you use `fetch` API or other libraries that don't reject on 4xx/5xx status codes, transform them yourself

```typescript
const queryFn = async () => {
  const response = await fetch('/api/todos')
  if (!response.ok) {
    throw new Error('Failed to fetch')  // ✅ Transform to rejected Promise
  }
  return response.json()
}
```

## Don't Do

- **Don't use `onError` on custom hooks for toasts**
  - If you use `onError` in a custom hook, it will fire for **every Observer** (every component using that hook)
  - This means multiple toasts for a single failed request
  - Fix: Use global QueryCache callbacks instead

- **Don't unmount UI on background errors**
  - Don't unmount your complete UI just because a background refetch failed
  - Fix: Show error toasts for background refetches while keeping stale UI visible

- **Don't forget to transform errors for fetch API**
  - React Query needs a **rejected Promise** for error handling
  - Libraries like `axios` provide this automatically
  - `fetch` API requires manual transformation
