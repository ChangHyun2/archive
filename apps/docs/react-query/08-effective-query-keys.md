# Effective React Query Keys - Best Practices

## Best Practices

### 1. Colocate Query Keys

Keep Query Keys next to their respective queries, co-located in a feature directory. Don't store all Query Keys globally in `/src/utils/queryKeys.ts`.

**Recommended structure:**
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

The `queries.ts` file contains everything React Query related. Usually only export custom hooks, keeping Query Functions and Query Keys local.

### 2. Always Use Array Keys

Always use arrays for Query Keys, even though strings are technically allowed. React Query will internally convert them to arrays anyway.

```typescript
// 🚨 Will be transformed to ['todos'] anyhow
useQuery({ queryKey: 'todos' })

// ✅
useQuery({ queryKey: ['todos'] })
```

**Update**: With React Query v4, all keys need to be Arrays.

### 3. Structure Keys from Generic to Specific

Structure your Query Keys from most generic to most specific, with as many levels of granularity as needed:

```typescript
['todos', 'list', { filters: 'all' }]
['todos', 'list', { filters: 'done' }]
['todos', 'detail', 1]
['todos', 'detail', 2]
```

This structure allows you to:
- Invalidate everything todo-related with `['todos']`
- Target all lists or all details
- Target one specific list if you know the exact key

### 4. Use Query Key Factories

Use a Query Key factory per feature to avoid manual key declarations (error-prone and hard to change):

```typescript
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
}
```

**Usage examples:**
```typescript
// Remove everything related to the todos feature
queryClient.removeQueries({
  queryKey: todoKeys.all
})

// Invalidate all the lists
queryClient.invalidateQueries({
  queryKey: todoKeys.lists()
})

// Prefetch a single todo
queryClient.prefetchQueries({
  queryKey: todoKeys.detail(id),
  queryFn: () => fetchTodo(id),
})
```

### 5. Queries are Declarative - Use Query Key for Parameters

**Don't use `refetch` to pass parameters.** Instead, put state in the Query Key and let React Query automatically refetch when the key changes.

```typescript
// ❌ Don't do this
function Component() {
  const { data, refetch } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  })
  
  // ❓ How do I pass parameters to refetch?
  return <Filters onApply={() => refetch(???)} />
}

// ✅ Do this instead
function Component() {
  const [filters, setFilters] = React.useState()
  const { data } = useQuery({
    queryKey: ['todos', filters],  // State in query key
    queryFn: () => fetchTodos(filters),
  })
  
  // ✅ Set local state and let it drive the query
  return <Filters onApply={setFilters} />
}
```

The re-render triggered by `setFilters` will pass a different Query Key to React Query, which will make it refetch automatically.

### 6. Flexible Cache Updates from Mutations

With a well-structured key hierarchy, you can update cache flexibly:

```typescript
function useUpdateTitle() {
  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      // ✅ Update the todo detail
      queryClient.setQueryData(
        ['todos', 'detail', newTodo.id],
        newTodo
      )

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

Or invalidate all lists:
```typescript
function useUpdateTitle() {
  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      queryClient.setQueryData(
        ['todos', 'detail', newTodo.id],
        newTodo
      )

      // ✅ Just invalidate all the lists
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list']
      })
    },
  })
}
```

Or combine both approaches:
```typescript
function useUpdateTitle() {
  const { filters } = useFilterParams()

  return useMutation({
    mutationFn: updateTitle,
    onSuccess: (newTodo) => {
      queryClient.setQueryData(
        ['todos', 'detail', newTodo.id],
        newTodo
      )

      // ✅ Update the list we are currently on
      queryClient.setQueryData(
        ['todos', 'list', { filters }],
        (previous) =>
          previous.map((todo) =>
            todo.id === newTodo.id ? newTodo : todo
          )
      )

      // ✅ Invalidate all the lists, but don't refetch the active one
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list'],
        refetchType: 'none',  // v4+ (was refetchActive: false in v3)
      })
    },
  })
}
```

## Not To Do

### 1. Don't Store All Query Keys Globally

Storing all Query Keys in `/src/utils/queryKeys.ts` doesn't scale well. Colocate them with their queries instead.

### 2. Don't Use String Keys

Always use arrays, even though strings work. React Query v4+ requires arrays.

### 3. Don't Use refetch to Pass Parameters

`refetch` is for refetching with the same parameters. If you need different parameters, put them in the Query Key instead.

### 4. Don't Use Same Key for useQuery and useInfiniteQuery

```typescript
// 🚨 This won't work
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})

useInfiniteQuery({
  queryKey: ['todos'],  // ❌ Same key!
  queryFn: fetchInfiniteTodos,
})

// ✅ Choose something else instead
useInfiniteQuery({
  queryKey: ['infiniteTodos'],  // ✅ Different key
  queryFn: fetchInfiniteTodos,
})
```

There is only one Query Cache, and you would share data between these two, which is not good because infinite queries have a fundamentally different structure than normal queries.
