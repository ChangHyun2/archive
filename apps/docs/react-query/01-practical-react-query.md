# Practical React Query - Best Practices

## Best Practices

### 1. Treat the query key like a dependency array

Always include variables used in `queryFn` as part of the `queryKey`. React Query will automatically trigger a refetch when the query key changes, similar to how `useEffect` works with dependency arrays.

```typescript
type State = 'all' | 'open' | 'done'
type Todo = {
  id: number
  state: State
}
type Todos = ReadonlyArray<Todo>

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`)
  return response.data
}

export const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos', state],  // state is in queryKey
    queryFn: () => fetchTodos(state),
  })
```

**Key point**: Never pass a variable to `queryFn` that is not part of the `queryKey`.

### 2. Use initialData to improve UX

When switching between query keys (e.g., filtering), use `initialData` to pre-fill cache entries and avoid hard loading states.

```typescript
export const useTodosQuery = (state: State) =>
  useQuery({
    queryKey: ['todos', state],
    queryFn: () => fetchTodos(state),
    initialData: () => {
      const allTodos = queryClient.getQueryData<Todos>([
        'todos',
        'all',
      ])
      const filteredData =
        allTodos?.filter((todo) => todo.state === state) ?? []

      return filteredData.length > 0 ? filteredData : undefined
    },
  })
```

This allows instant display of filtered data while background fetch completes.

### 3. Keep server and client state separate

**Do NOT** put data from `useQuery` into local state. This opts you out of all background updates that React Query provides.

**Exception**: Only acceptable when fetching default values for forms, and in that case, set `staleTime: Infinity` to prevent unnecessary refetches:

```typescript
const App = () => {
  const { data } = useQuery({
    queryKey: ['key'],
    queryFn,
    staleTime: Infinity,  // Prevent background refetches
  })

  return data ? <MyForm initialData={data} /> : null
}

const MyForm = ({ initialData }) => {
  const [data, setData] = React.useState(initialData)
  // ...
}
```

### 4. Use the `enabled` option for powerful patterns

The `enabled` option enables several useful patterns:
- **Dependent Queries**: Fetch data in one query, run second query only after first succeeds
- **Turn queries on/off**: Pause polling when modal is open
- **Wait for user input**: Disable query until filters are applied
- **Disable after user input**: Use draft values that take precedence over server data

### 5. Create custom hooks

Even for simple queries, creating custom hooks pays off:
- Keeps data fetching logic out of UI but co-located with `useQuery`
- Centralizes all usages of a query key (and type definitions)
- Single place to tweak settings or add data transformations

### 6. Use React Query DevTools

The DevTools help immensely in understanding query state and debugging. Throttle network connection in browser DevTools to better recognize background refetches.

### 7. Understand staleTime vs gcTime

- **staleTime**: Duration until query transitions from fresh to stale. While fresh, data is read from cache only (no network request).
- **gcTime**: Duration until inactive queries are removed from cache (default: 5 minutes). Queries become inactive when all components using them unmount.

Most of the time, you'll need to adjust `staleTime`, rarely `gcTime`.

## Not To Do

### 1. Don't use queryCache as a local state manager

If you use `queryClient.setQueryData`, it should only be for:
- Optimistic updates
- Writing data received from backend after mutations

**Remember**: Every background refetch might override that data, so use something else for local state.

### 2. Don't put useQuery data into local state

This implicitly opts you out of all background updates. The state "copy" will not update with background refetches.

**Exception**: Only acceptable for form initial data with `staleTime: Infinity`.
