# The Query Options API - Best Practices

## Key Concept

React Query v5 introduced a unified API: **all functions now accept a single options object** instead of multiple arguments.

## Best Practices

### 1. Use Query Options Object Pattern

**Best Practice**: Always use the object API (v5+):

```typescript
// ✅ v5+ API - single object
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 5000
})

// ✅ Also works for imperative calls
queryClient.invalidateQueries({ queryKey: ['todos'] })
```

**Why**:
- Simpler, more consistent API
- Easier to understand for new users
- Extensible - easy to add new options
- Better for sharing options between functions

### 2. Abstract Query Options for Reusability

**Best Practice**: Extract query options into a reusable object:

```typescript
const todosQuery = {
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 5000,
}

// ✅ Works everywhere
useQuery(todosQuery)
queryClient.prefetchQuery(todosQuery)
useSuspenseQuery(todosQuery)
useQueries([{ queries: [todosQuery] }])
```

**Benefits**:
- Share options between different functions
- Works with imperative calls (prefetch, invalidate)
- Single source of truth for query configuration

### 3. Use `queryOptions` Helper for Type Safety

**Best Practice**: Use `queryOptions` helper to catch typos and enable type inference:

```typescript
// ✅ Type-safe with queryOptions
const todosQuery = queryOptions({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 5000,  // ✅ Typo would be caught
})

// 🚨 This would error with queryOptions
const todosQuery = queryOptions({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  stallTime: 5000,  // ❌ TypeScript error!
})
```

**Why**: 
- TypeScript catches typos when using `queryOptions`
- Without it, typos in abstracted objects are silently ignored
- Enables type inference for `getQueryData`, `setQueryData`, etc.

### 4. Leverage DataTag for Type-Safe Cache Access

**Best Practice**: Use `queryOptions` to get type-safe cache access:

```typescript
const todosQuery = queryOptions({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 5000,
})

// ✅ Type inferred automatically!
const todos = queryClient.getQueryData(todosQuery.queryKey)
//    ^? const todos: Todo[] | undefined

// ❌ Old way - manual type parameter
const todos = queryClient.getQueryData<Array<Todo>>(['todos'])
```

**How it works**: `queryOptions` "tags" the `queryKey` with type information from `queryFn`. Functions like `getQueryData` and `setQueryData` can read this tag to infer types.

### 5. Combine Query Key Factories with Query Options

**Best Practice**: Create query factories that combine keys and functions:

```typescript
const todoQueries = {
  // Key-only entries for hierarchy and invalidation
  all: () => ['todos'],
  lists: () => [...todoQueries.all(), 'list'],
  details: () => [...todoQueries.all(), 'detail'],
  
  // Full query objects with queryOptions
  list: (filters: string) =>
    queryOptions({
      queryKey: [...todoQueries.lists(), filters],
      queryFn: () => fetchTodos(filters),
    }),
  detail: (id: number) =>
    queryOptions({
      queryKey: [...todoQueries.details(), id],
      queryFn: () => fetchTodo(id),
      staleTime: 5000,
    }),
}
```

**Benefits**:
- Type safety
- Co-location of `queryKey` and `queryFn`
- Great developer experience
- Single source of truth

**Key insight**: Separating `queryKey` from `queryFn` was a mistake - they should be co-located since the key defines dependencies for the function.

### 6. Reconsider Custom Hooks for Simple Cases

**Best Practice**: You might not need a custom hook if it's just a wrapper:

```typescript
// ✅ Direct usage is fine
const { data } = useQuery(todosQuery)

// ✅ Or useSuspenseQuery
const { data } = useSuspenseQuery(todosQuery)

// 🚨 Unnecessary wrapper
const useTodos = () => useQuery(todosQuery)
```

**When to use custom hooks**:
- When you need additional logic (e.g., `useMemo` for derived state)
- When you need to compose multiple queries
- When you need to add component-specific logic

**When not to use**:
- Simple wrappers that just call `useQuery`
- When you want flexibility to use `useSuspenseQuery` or `useQuery` interchangeably

## Not To Do

### 1. Don't Use Old Multi-Argument API (v5+)

```typescript
// 🚨 Old v3/v4 API - deprecated in v5
useQuery(
  ['todos'],
  fetchTodos,
  { staleTime: 5000 }
)

// ✅ Use object API
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 5000
})
```

### 2. Don't Abstract Options Without `queryOptions`

```typescript
// 🚨 Typo won't be caught
const todosQuery = {
  queryKey: ['todos'],
  queryFn: fetchTodos,
  stallTime: 5000,  // ❌ Typo, but no error!
}

useQuery(todosQuery)  // staleTime not applied

// ✅ Use queryOptions to catch typos
const todosQuery = queryOptions({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  stallTime: 5000,  // ✅ TypeScript error!
})
```

### 3. Don't Separate QueryKey from QueryFunction

```typescript
// 🚨 Separation makes it easy to get out of sync
const todoKeys = {
  list: (filters: string) => ['todos', 'list', filters],
}

// Somewhere else...
const useTodos = (filters: string) => {
  return useQuery({
    queryKey: todoKeys.list(filters),
    queryFn: () => fetchTodos(filters),  // Easy to forget to update
  })
}

// ✅ Co-locate with queryOptions
const todoQueries = {
  list: (filters: string) =>
    queryOptions({
      queryKey: ['todos', 'list', filters],
      queryFn: () => fetchTodos(filters),  // ✅ Always in sync
    }),
}
```

### 4. Don't Use Manual Type Parameters When You Have queryOptions

```typescript
// 🚨 Manual type parameter - not type-safe
const todos = queryClient.getQueryData<Array<Todo>>(['todos'])

// ✅ Automatic type inference
const todosQuery = queryOptions({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})

const todos = queryClient.getQueryData(todosQuery.queryKey)
//    ^? Type inferred: Todo[] | undefined
```

## Key Insights

1. **Unified API**: All functions accept single options object in v5+
2. **queryOptions helper**: Provides type safety and enables DataTag
3. **Abstract options**: Share query configuration between different functions
4. **DataTag magic**: Type-safe cache access without manual type parameters
5. **Co-location**: Keep `queryKey` and `queryFn` together in query factories
6. **Custom hooks**: Not always needed - direct usage is fine for simple cases
7. **Type safety**: `queryOptions` catches typos and enables better inference
