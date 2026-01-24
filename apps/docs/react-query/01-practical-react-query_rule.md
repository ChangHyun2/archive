---
description: "React Query practical usage patterns: query keys, initial data, state separation, enabled option, custom hooks, and staleTime vs gcTime"
alwaysApply: false
---

# React Query Practical Usage Rules

## Query Keys

- **Always include variables used in `queryFn` as part of the `queryKey`**
  - React Query automatically triggers refetch when query key changes
  - Never pass a variable to `queryFn` that is not part of the `queryKey`

```typescript
// ✅ Correct
const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos', state],  // state is in queryKey
    queryFn: () => fetchTodos(state),
  })

// ❌ Wrong - state not in queryKey
const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos'],
    queryFn: () => fetchTodos(state),  // state used but not in key
  })
```

## Initial Data

- **Use `initialData` to improve UX when switching between query keys**
  - Pre-fill cache entries to avoid hard loading states
  - Useful for filtering scenarios

```typescript
export const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos', state],
    queryFn: () => fetchTodos(state),
    initialData: () => {
      const allTodos = queryClient.getQueryData<Todos>(['todos', 'all'])
      const filteredData = allTodos?.filter((todo) => todo.state === state) ?? []
      return filteredData.length > 0 ? filteredData : undefined
    },
  })
```

## State Separation

- **Do NOT put data from `useQuery` into local state**
  - This opts you out of all background updates that React Query provides
  - Exception: Only acceptable when fetching default values for forms with `staleTime: Infinity`

```typescript
// ❌ Wrong - loses background updates
const { data } = useQuery({ queryKey: ['key'], queryFn })
const [localData, setLocalData] = useState(data)

// ✅ Correct - use data directly
const { data } = useQuery({ queryKey: ['key'], queryFn })

// ✅ Exception - form initial data
const { data } = useQuery({
  queryKey: ['key'],
  queryFn,
  staleTime: Infinity,  // Prevent background refetches
})
return data ? <MyForm initialData={data} /> : null
```

## Enabled Option

- **Use `enabled` option for powerful patterns:**
  - Dependent queries: Fetch data in one query, run second query only after first succeeds
  - Turn queries on/off: Pause polling when modal is open
  - Wait for user input: Disable query until filters are applied
  - Disable after user input: Use draft values that take precedence over server data

## Custom Hooks

- **Create custom hooks even for simple queries**
  - Keeps data fetching logic out of UI but co-located with `useQuery`
  - Centralizes all usages of a query key (and type definitions)
  - Single place to tweak settings or add data transformations

## staleTime vs gcTime

- **Understand the difference:**
  - `staleTime`: Duration until query transitions from fresh to stale. While fresh, data is read from cache only (no network request)
  - `gcTime`: Duration until inactive queries are removed from cache (default: 5 minutes). Queries become inactive when all components using them unmount
- **Most of the time, you'll need to adjust `staleTime`, rarely `gcTime`**

## Don't Do

- **Don't use `queryClient.setQueryData` as a local state manager**
  - Should only be used for:
    - Optimistic updates
    - Writing data received from backend after mutations
  - Every background refetch might override that data

- **Don't put `useQuery` data into local state**
  - This implicitly opts you out of all background updates
  - Exception: Only acceptable for form initial data with `staleTime: Infinity`
