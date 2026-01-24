---
description: "React Query Query Options API (v5+): use queryOptions helper, abstract options for reusability, combine with query key factories"
alwaysApply: false
---

# React Query Query Options API Rules

## Use Query Options Object Pattern (v5+)

- **Always use the object API (v5+)**

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

## Abstract Query Options for Reusability

- **Extract query options into a reusable object**

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

- **Benefits**:
  - Share options between different functions
  - Works with imperative calls (prefetch, invalidate)
  - Single source of truth for query configuration

## Use queryOptions Helper for Type Safety

- **Use `queryOptions` helper to catch typos and enable type inference**

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

- **Why**: TypeScript catches typos when using `queryOptions`. Without it, typos in abstracted objects are silently ignored.

## Leverage DataTag for Type-Safe Cache Access

- **Use `queryOptions` to get type-safe cache access**

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

- **How it works**: `queryOptions` "tags" the `queryKey` with type information from `queryFn`. Functions like `getQueryData` and `setQueryData` can read this tag to infer types.

## Combine Query Key Factories with Query Options

- **Create query factories that combine keys and functions**

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

- **Benefits**:
  - Type safety
  - Co-location of `queryKey` and `queryFn`
  - Great developer experience
  - Single source of truth

## Reconsider Custom Hooks for Simple Cases

- **You might not need a custom hook if it's just a wrapper**

```typescript
// ✅ Direct usage is fine
const { data } = useQuery(todosQuery)

// ✅ Or useSuspenseQuery
const { data } = useSuspenseQuery(todosQuery)

// 🚨 Unnecessary wrapper
const useTodos = () => useQuery(todosQuery)
```

- **When to use custom hooks**:
  - When you need additional logic (e.g., `useMemo` for derived state)
  - When you need to compose multiple queries
  - When you want to abstract away React Query from components

## Don't Do

- **Don't use old API patterns (v4 and below)**
  - v5+ uses unified object API
  - Fix: Always use object API for consistency

- **Don't abstract options without `queryOptions`**
  - Typos are silently ignored
  - Fix: Use `queryOptions` helper for type safety

- **Don't separate `queryKey` from `queryFn` unnecessarily**
  - They should be co-located since the key defines dependencies for the function
  - Fix: Combine in query factories with `queryOptions`
