# Leveraging the Query Function Context - Best Practices

## Key Concept

**Hot take**: Don't use inline functions - leverage the Query Function Context given to you, and use a Query Key factory that produces object keys.

## Best Practices

### 1. Use QueryFunctionContext Instead of Closures

**Best Practice**: Get parameters from `queryKey` in the `QueryFunctionContext` instead of closing over variables:

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

**Why**: This ensures you can't use parameters in `queryFn` without adding them to `queryKey`. Prevents queryKey from getting out of sync with dependencies.

### 2. Use Query Key Factories with Type Safety

**Best Practice**: Use a typed query key factory and type the `QueryFunctionContext`:

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

export const useTodos = () => {
  const { state, sorting } = useTodoParams()

  return useQuery({
    queryKey: todoKeys.list(state, sorting),
    queryFn: fetchTodos
  })
}
```

**Benefits**:
- Type safety: Only accepts keys from the factory
- Prevents using wrong query keys
- Ensures queryKey and queryFn stay in sync

### 3. Use Object Query Keys for Better Destructuring

**Best Practice**: Use object keys instead of arrays for named destructuring:

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

**Benefits**:
- Named destructuring (no need to skip array indices)
- More powerful fuzzy matching (order doesn't matter)
- Less error-prone when adding new scopes

### 4. Leverage Object Keys for Flexible Fuzzy Matching

**Best Practice**: Use object keys for more powerful invalidation patterns:

```typescript
// 🕺 Remove everything related to the todos feature
queryClient.removeQueries({
  queryKey: [{ scope: 'todos' }]
})

// 🚀 Reset all todo lists
queryClient.resetQueries({
  queryKey: [{ scope: 'todos', entity: 'list' }]
})

// 🙌 Invalidate all lists across all scopes
queryClient.invalidateQueries({
  queryKey: [{ entity: 'list' }]
})
```

**Why**: Objects have no order, so you can match by any property combination, not just prefix matching.

## Not To Do

### 1. Don't Use Inline Functions with Closures

```typescript
// 🚨 Inline function closes over state
export const useTodos = () => {
  const { state } = useTodoParams()

  return useQuery({
    queryKey: ['todos', state],
    queryFn: () => fetchTodos(state),  // ❌ Closes over state
  })
}
```

**Problem**: When you add more parameters, queryKey can get out of sync:

```typescript
// 🚨 queryKey missing sorting parameter!
export const useTodos = () => {
  const { state, sorting } = useTodoParams()

  return useQuery({
    queryKey: ['todos', state],  // ❌ Missing sorting
    queryFn: () => fetchTodos(state, sorting),  // Uses sorting but not in key
  })
}
```

**Result**: Changing sorting doesn't trigger a refetch because it's not in the queryKey.

### 2. Don't Destructure Array Keys by Index

```typescript
// 🚨 Weird destructuring - skipping first two parts
const [, , state, sorting] = queryKey
```

**Problem**: 
- Hard to read
- Error-prone when adding new scopes
- Easy to get indices wrong

**Solution**: Use object keys with named destructuring.

### 3. Don't Build URLs Unsafely from Array Keys

```typescript
// 🚨 Unsafe - can stringify everything
queryFn: async ({ queryKey }) => {
  const response = await axios.get(
    `todos/${queryKey[1]}?sorting=${queryKey[2]}`  // ❌ No type safety
  )
  return response.data
}
```

**Problem**: No guarantee that `queryKey[1]` and `queryKey[2]` are the right types or even exist.

**Solution**: Use typed query key factories with `QueryFunctionContext`.

## Key Insights

1. **QueryFunctionContext prevents sync issues**: Using `queryKey` from context ensures it can't diverge from dependencies
2. **Type safety matters**: Typed query key factories + `QueryFunctionContext` provide compile-time safety
3. **Object keys are better**: Named destructuring and flexible matching make them superior to arrays
4. **Tradeoff exists**: More complexity for better type safety - choose based on team size and needs
5. **ESLint rule exists**: There's now a lint rule to catch these issues automatically
