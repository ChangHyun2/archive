---
description: "React Query key patterns: colocate keys, use arrays, structure from generic to specific, use factories, treat as dependencies"
alwaysApply: false
---

# React Query Query Keys Rules

## Colocate Query Keys

- **Keep Query Keys next to their respective queries, co-located in a feature directory**
  - Don't store all Query Keys globally in `/src/utils/queryKeys.ts`
  - Recommended structure: feature directory with `queries.ts` file

```
- src
  - features
    - Profile
      - index.tsx
      - queries.ts
    - Todos
      - index.tsx
      - queries.ts
```

## Always Use Array Keys

- **Always use arrays for Query Keys**
  - React Query v4+ requires arrays
  - Even though strings work in v3, they're converted to arrays anyway

```typescript
// ✅ Correct
useQuery({ queryKey: ['todos'] })

// ❌ Wrong (v3) / Invalid (v4+)
useQuery({ queryKey: 'todos' })
```

## Structure Keys from Generic to Specific

- **Structure your Query Keys from most generic to most specific**

```typescript
['todos', 'list', { filters: 'all' }]
['todos', 'list', { filters: 'done' }]
['todos', 'detail', 1]
['todos', 'detail', 2]
```

- This structure allows you to:
  - Invalidate everything todo-related with `['todos']`
  - Target all lists or all details
  - Target one specific list if you know the exact key

## Use Query Key Factories

- **Use a Query Key factory per feature to avoid manual key declarations**

```typescript
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
}
```

- **Usage examples:**

```typescript
// Remove everything related to the todos feature
queryClient.removeQueries({ queryKey: todoKeys.all })

// Invalidate all the lists
queryClient.invalidateQueries({ queryKey: todoKeys.lists() })

// Prefetch a single todo
queryClient.prefetchQueries({
  queryKey: todoKeys.detail(id),
  queryFn: () => fetchTodo(id),
})
```

## Queries are Declarative - Use Query Key for Parameters

- **Don't use `refetch` to pass parameters**
  - Put state in the Query Key and let React Query automatically refetch when the key changes

```typescript
// ❌ Wrong - how do I pass parameters to refetch?
function Component() {
  const { data, refetch } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })
  return <Filters onApply={() => refetch(???)} />
}

// ✅ Correct - state in query key
function Component() {
  const [filters, setFilters] = React.useState()
  const { data } = useQuery({
    queryKey: ['todos', filters],  // State in query key
    queryFn: () => fetchTodos(filters),
  })
  return <Filters onApply={setFilters} />
}
```

## Flexible Cache Updates from Mutations

- **With a well-structured key hierarchy, you can update cache flexibly:**

```typescript
function useUpdateTitle() {
  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      // ✅ Update the todo detail
      queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo)

      // ✅ Update all the lists that contain this todo
      queryClient.setQueriesData(['todos', 'list'], (previous) =>
        previous.map((todo) =>
          todo.id === newTodo.id ? newTodo : todo
        )
      )
    },
  })
}
```

## Don't Do

- **Don't store all Query Keys globally**
  - Storing all Query Keys in `/src/utils/queryKeys.ts` doesn't scale well
  - Colocate them with their queries instead

- **Don't use string keys**
  - Always use arrays, React Query v4+ requires arrays

- **Don't use `refetch` to pass parameters**
  - `refetch` is for refetching with the same parameters
  - If you need different parameters, put them in the Query Key instead

- **Don't use same key for `useQuery` and `useInfiniteQuery`**
  - There is only one Query Cache, and you would share data between these two
  - Infinite queries have a fundamentally different structure than normal queries

```typescript
// ❌ Wrong
useQuery({ queryKey: ['todos'], queryFn: fetchTodos })
useInfiniteQuery({ queryKey: ['todos'], queryFn: fetchInfiniteTodos })

// ✅ Correct
useQuery({ queryKey: ['todos'], queryFn: fetchTodos })
useInfiniteQuery({ queryKey: ['infiniteTodos'], queryFn: fetchInfiniteTodos })
```
