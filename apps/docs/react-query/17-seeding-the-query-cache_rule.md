---
description: "React Query cache seeding: prefetch queries before render, use useSuspenseQueries, seed detail from list with initialData and initialDataUpdatedAt"
alwaysApply: false
---

# React Query Cache Seeding Rules

## Prefetch Queries Before Component Renders

- **Use `prefetchQuery` to initiate fetches early**

```typescript
const issuesQuery = { queryKey: ['issues'], queryFn: fetchIssues }

// ✅ Initiate fetch before component renders
queryClient.prefetchQuery(issuesQuery)

function Issues() {
  const issues = useSuspenseQuery(issuesQuery)  // ✅ Data already fetching
}
```

- **For multiple queries**: Prefetch all of them in parallel

```typescript
// ✅ Prefetch both in parallel
queryClient.prefetchQuery(issuesQuery)
queryClient.prefetchQuery(labelsQuery)

function Component() {
  const issues = useSuspenseQuery(issuesQuery)
  const labels = useSuspenseQuery(labelsQuery)  // ✅ Both already fetching
}
```

## Use useSuspenseQueries (v5+)

- **Use dedicated hook for parallel suspense queries**

```typescript
// ✅ v5+ - triggers all fetches in parallel
const [issues, labels] = useSuspenseQueries({
  queries: [
    { queryKey: ['issues'], queryFn: fetchIssues },
    { queryKey: ['labels'], queryFn: fetchLabels },
  ],
})
```

- **Why**: Automatically triggers all fetches in parallel, avoiding waterfalls.

## Seed Detail Cache from List Cache (Pull Approach)

- **Use `initialData` to pull data from list cache**

```typescript
const useTodo = (id: number) => {
  const queryClient = useQueryClient()
  return useQuery({
    queryKey: ['todos', 'detail', id],
    queryFn: () => fetchTodo(id),
    initialData: () => {
      // ✅ Look up list cache for the item
      return queryClient
        .getQueryData(['todos', 'list'])
        ?.find((todo) => todo.id === id)
    },
    initialDataUpdatedAt: () =>
      // ✅ Get last fetch time of the list
      queryClient.getQueryState(['todos', 'list'])?.dataUpdatedAt,
  })
}
```

- **Why**: Seeds cache "just in time". Without `initialDataUpdatedAt`, data appears fresh even if list is stale.

## Seed Detail Cache When Fetching List (Push Approach)

- **Create detail caches when fetching list**

```typescript
const useTodos = () => {
  const queryClient = useQueryClient()
  return useQuery({
    queryKey: ['todos', 'list'],
    queryFn: async () => {
      const todos = await fetchTodos()
      todos.forEach((todo) => {
        // ✅ Create detail cache for each item
        queryClient.setQueryData(['todos', 'detail', todo.id], todo)
      })
      return todos
    },
  })
}
```

- **Tradeoffs**:
  - ✅ `staleTime` automatically respected
  - ❌ Might create unnecessary cache entries
  - ❌ Pushed data might be garbage collected too early

## Don't Do

- **Don't use multiple suspense queries without prefetching**
  - Creates waterfall
  - Fix: Prefetch both, or use `useSuspenseQueries`

- **Don't forget `initialDataUpdatedAt` when seeding**
  - Data appears fresh even if list is stale
  - Fix: Add `initialDataUpdatedAt` to respect list's staleness

- **Don't use push approach for large lists**
  - Creates many unnecessary cache entries
  - Fix: Use pull approach for large lists

- **Don't seed if data structures don't match**
  - List item missing required field
  - Fix: Use `placeholderData` instead if structures don't match
