# React Query Data Transformations - Best Practices

## Best Practices

### 1. Transform on the backend (if possible)

**Best approach**: If the backend returns data in exactly the structure you want, there's nothing to do on the frontend.

✅ No work on the frontend  
❌ Not always possible (e.g., public REST APIs)

### 2. Transform in the queryFn

Transform data before it enters the cache. The transformed structure winds up in the cache, so you won't have access to the original structure.

```typescript
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos')
  const data: Todos = response.data

  return data.map((todo) => todo.name.toUpperCase())
}

export const useTodosQuery = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })
```

**Pros:**
- Very "close to the backend" in terms of co-location
- Works with data "as if it came like this from the backend"

**Cons:**
- Transformed structure in cache (no access to original)
- Runs on every fetch (no optimization)
- Not feasible if you have a shared API layer you cannot modify

**Note**: DevTools will show transformed structure, network trace shows original - this might be confusing.

### 3. Transform in render function (with useMemo)

Transform in custom hooks, but be careful with dependencies:

```typescript
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos')
  return response.data
}

export const useTodosQuery = () => {
  const queryInfo = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })

  return {
    ...queryInfo,
    // ✅ Correctly memoizes by queryInfo.data
    data: React.useMemo(
      () => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
      [queryInfo.data]  // Use data, not queryInfo!
    ),
  }
}
```

**Pros:**
- Optimizable via useMemo
- Good if you have additional logic in custom hook

**Cons:**
- More convoluted syntax
- Data can be potentially undefined (use optional chaining)
- **Not recommended with tracked queries** (v4+): Spreading `...queryInfo` invokes getters on all properties

### 4. Use the `select` option (Recommended)

The `select` option is the best approach for data transformations:

```typescript
export const useTodosQuery = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select: (data) => data.map((todo) => todo.name.toUpperCase()),
  })
```

**Memoize expensive transformations:**

```typescript
// Option 1: Extract to stable function reference
const transformTodoNames = (data: Todos) =>
  data.map((todo) => todo.name.toUpperCase())

export const useTodosQuery = () =>
  useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    select: transformTodoNames,  // ✅ Stable reference
  })

// Option 2: Use useCallback
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

**Partial subscriptions** - Subscribe to only parts of the data:

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

Components using `useTodosCount` won't rerender when individual todos change - only when the count changes!

**Pros:**
- ✅ Best optimizations
- ✅ Allows for partial subscriptions
- ✅ Selectors only called if data exists (no undefined handling needed)

**Cons:**
- Structure can be different for every observer
- Structural sharing is performed twice

## Not To Do

### 1. Don't use queryInfo as useMemo dependency

```typescript
// 🚨 Don't do this - the useMemo does nothing at all here!
data: React.useMemo(
  () => queryInfo.data?.map((todo) => todo.name.toUpperCase()),
  [queryInfo]  // ❌ This will cause recomputation on every render
)
```

**Why**: `queryInfo` itself is not referentially stable, so the memoization is useless. Use `queryInfo.data` instead.

### 2. Don't spread queryInfo with tracked queries (v4+)

Since React Query v4 has tracked queries enabled by default, spreading `...queryInfo` is no longer recommended because it invokes getters on all properties.
