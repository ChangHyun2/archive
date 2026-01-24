---
description: "React Query QueryFunctionContext: use context instead of closures, use typed query key factories, prefer object keys for destructuring"
alwaysApply: false
---

# React Query Query Function Context Rules

## Use QueryFunctionContext Instead of Closures

- **Get parameters from `queryKey` in the `QueryFunctionContext` instead of closing over variables**

```typescript
// ✅ Use QueryFunctionContext
const fetchTodos = async ({ queryKey }) => {
  // Get all params from the queryKey
  const [, state, sorting] = queryKey
  const response = await axios.get(`todos/${state}?sorting=${sorting}`)
  return response.data
}

export const useTodos = () => {
  const { state, sorting } = useTodoParams()

  // ✅ No need to pass parameters manually
  return useQuery({
    queryKey: ['todos', state, sorting],
    queryFn: fetchTodos,
  })
}
```

- **Why**: This ensures you can't use parameters in `queryFn` without adding them to `queryKey`. Prevents queryKey from getting out of sync with dependencies.

## Use Query Key Factories with Type Safety

- **Use a typed query key factory and type the `QueryFunctionContext`**

```typescript
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (state: State, sorting: Sorting) =>
    [...todoKeys.lists(), state, sorting] as const,
}

const fetchTodos = async ({
  queryKey,
}: QueryFunctionContext<ReturnType<typeof todoKeys['list']>>) => {
  const [, , state, sorting] = queryKey
  const response = await axios.get(`todos/${state}?sorting=${sorting}`)
  return response.data
}
```

## Use Object Query Keys for Better Destructuring

- **Use object keys instead of arrays for named destructuring**

```typescript
const todoKeys = {
  // ✅ All keys are arrays with exactly one object
  all: [{ scope: 'todos' }] as const,
  lists: () => [{ ...todoKeys.all[0], entity: 'list' }] as const,
  list: (state: State, sorting: Sorting) =>
    [{ ...todoKeys.lists()[0], state, sorting }] as const,
}

const fetchTodos = async ({
  // ✅ Extract named properties from the queryKey
  queryKey: [{ state, sorting }],
}: QueryFunctionContext<ReturnType<typeof todoKeys['list']>>) => {
  const response = await axios.get(`todos/${state}?sorting=${sorting}`)
  return response.data
}
```

- **Benefits:**
  - Named destructuring (no need to skip array indices)
  - More powerful fuzzy matching (order doesn't matter)
  - Less error-prone when adding new scopes

## Don't Do

- **Don't use inline functions with closures**
  - When you add more parameters, queryKey can get out of sync
  - Result: Changing parameters doesn't trigger a refetch because it's not in the queryKey

```typescript
// ❌ Wrong - inline function closes over state
export const useTodos = () => {
  const { state, sorting } = useTodoParams()

  return useQuery({
    queryKey: ['todos', state],  // ❌ Missing sorting
    queryFn: () => fetchTodos(state, sorting),  // Uses sorting but not in key
  })
}
```

- **Don't destructure array keys by index**
  - Hard to read
  - Error-prone when adding new scopes
  - Easy to get indices wrong
  - Fix: Use object keys with named destructuring

- **Don't build URLs unsafely from array keys**
  - No guarantee that `queryKey[1]` and `queryKey[2]` are the right types or even exist
  - Fix: Use typed query key factories with `QueryFunctionContext`
