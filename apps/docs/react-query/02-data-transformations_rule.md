---
description: "React Query data transformation patterns: use select option for transformations, avoid useMemo with queryInfo dependency"
alwaysApply: false
---

# React Query Data Transformations Rules

## Use `select` Option (Recommended)

- **Use the `select` option for data transformations**
  - Best optimizations
  - Allows for partial subscriptions
  - Selectors only called if data exists (no undefined handling needed)

```typescript
// ✅ Recommended
export const useTodosQuery = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select: (data) => data.map((todo) => todo.name.toUpperCase()),
  })
```

- **Memoize expensive transformations:**
  - Extract to stable function reference, or
  - Use `useCallback` with empty dependency array

```typescript
// ✅ Option 1: Extract to stable function
const transformTodoNames = (data: Todos) =>
  data.map((todo) => todo.name.toUpperCase())

export const useTodosQuery = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select: transformTodoNames,  // ✅ Stable reference
  })

// ✅ Option 2: Use useCallback
export const useTodosQuery = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select: React.useCallback(
      (data: Todos) => data.map((todo) => todo.name.toUpperCase()),
      []
    ),
  })
```

- **Partial subscriptions** - Subscribe to only parts of the data:

```typescript
export const useTodosQuery = (select) =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select,
  })

export const useTodosCount = () =>
  useTodosQuery((data) => data.length)

export const useTodo = (id) =>
  useTodosQuery((data) => data.find((todo) => todo.id === id))
```

## Transform in queryFn (Alternative)

- **Transform data before it enters the cache**
  - Transformed structure winds up in the cache
  - Works with data "as if it came like this from the backend"
  - Note: DevTools will show transformed structure, network trace shows original

```typescript
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos')
  const data: Todos = response.data
  return data.map((todo) => todo.name.toUpperCase())
}
```

## Don't Do

- **Don't use `queryInfo` as `useMemo` dependency**
  - `queryInfo` itself is not referentially stable, so the memoization is useless
  - Use `queryInfo.data` instead

```typescript
// ❌ Wrong - useMemo does nothing
data: React.useMemo(
  () => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
  [queryInfo]  // ❌ This will cause recomputation on every render
)

// ✅ Correct
data: React.useMemo(
  () => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
  [queryInfo.data]  // ✅ Use data, not queryInfo!
)
```

- **Don't spread `queryInfo` with tracked queries (v4+)**
  - Since React Query v4 has tracked queries enabled by default, spreading `...queryInfo` is no longer recommended because it invokes getters on all properties
