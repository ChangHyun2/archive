# React Query FAQs - Best Practices

## Common Questions and Best Practices

### 1. How to Pass Parameters to refetch?

**Answer**: You cannot pass parameters to `refetch`. Parameters are dependencies to your query - they should be in the `queryKey`.

**Best Practice**: Use declarative approach - put parameters in `queryKey`:

```typescript
// ✅ Declarative approach
const [id, setId] = useState(1)

const { data } = useQuery({
  queryKey: ['item', id],  // ✅ id is in queryKey
  queryFn: () => fetchItem({ id }),
})

<button onClick={() => setId(2)}>Show Item 2</button>
```

**Why**: 
- Different parameters = different cache entries
- React Query automatically fetches when `queryKey` changes
- No need to manually call `refetch`
- Works with URL state, zustand, redux, etc.

**Better**: Use URL state for shareable links:

```typescript
const { id } = useParams()

const { data } = useQuery({
  queryKey: ['item', id],
  queryFn: () => fetchItem({ id }),
})

// ✅ Change URL, React Query picks it up automatically
<Link to="/2">Show Item 2</Link>
```

**For smooth transitions**: Use `keepPreviousData`:

```typescript
import { keepPreviousData } from '@tanstack/react-query'

const { data, isPlaceholderData } = useQuery({
  queryKey: ['item', id],
  queryFn: () => fetchItem({ id }),
  placeholderData: keepPreviousData,  // ✅ Shows previous data while loading
})
```

### 2. Why Are Updates Not Shown?

**Common Issues**:

#### Issue 1: Query Keys Not Matching

**Best Practice**: Ensure exact key matching (type matters):

```typescript
// 🚨 These don't match!
['item', '1']  // string
['item', 1]    // number

// ✅ Use consistent types
const id = Number(useParams().id)  // Convert if needed
queryKey: ['item', id]
```

**Solution**: Use TypeScript and Query Key Factories to prevent this.

#### Issue 2: QueryClient Not Stable

**Best Practice**: Create `QueryClient` outside component or use stable reference:

```typescript
// ✅ Created outside App
const queryClient = new QueryClient()

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

**If you must create inside App**:

```typescript
// ✅ Stable with useState
export default function App() {
  const [queryClient] = React.useState(() => new QueryClient())
  
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

**Why**: New `QueryClient` = new empty cache. Unstable client means cache is lost on re-renders.

### 3. Why Should I Use `useQueryClient()` Instead of Importing?

**Best Practice**: Always use `useQueryClient()` hook:

```typescript
// ✅ Use hook
const queryClient = useQueryClient()
```

**Reasons**:

1. **Consistency**: `useQuery` uses `useQueryClient` internally - same client everywhere
2. **Decoupling**: Works with any client in context (useful for testing with different defaults)
3. **SSR/Microfrontends**: Client might be created inside App (can't export)
4. **Hooks in defaults**: If you need other hooks in `QueryClient` defaults, must create inside App

**Example with hooks in defaults**:

```typescript
export default function App() {
  const toast = useToast()  // ✅ Need hook here
  const [queryClient] = React.useState(
    () =>
      new QueryClient({
        mutationCache: new MutationCache({
          onError: (error) => toast.show({ type: 'error', error }),
        }),
      })
  )
  
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

**For class components**: Use render props pattern:

```typescript
const UseQueryClient = ({ children }) => children(useQueryClient())

<UseQueryClient>
  {(queryClient) => (
    <button onClick={() => queryClient.invalidateQueries({ queryKey: ['items'] })}>
      invalidate items
    </button>
  )}
</UseQueryClient>
```

### 4. Why Do I Not Get Errors?

**Common Issues**:

#### Issue 1: Fetch API Doesn't Throw on 4xx/5xx

**Best Practice**: Check `response.ok` and throw:

```typescript
// ✅ Transform 4xx/5xx into failed Promises
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    const response = await fetch('/todos/' + todoId)
    if (!response.ok) {
      throw new Error('Network response was not ok')
    }
    return response.json()
  },
})
```

**Why**: `fetch` only rejects on network errors, not HTTP error status codes.

#### Issue 2: Errors Caught Without Re-throwing

**Best Practice**: Always re-throw errors after logging:

```typescript
// ✅ Re-throw after logging
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    try {
      const { data } = await axios.get('/todos/' + todoId)
      return data
    } catch (error) {
      console.error(error)
      throw error  // ✅ Re-throw!
    }
  },
})
```

**Alternative**: Use global `onError` callback in `QueryCache` (see #11: React Query Error Handling).

### 5. Why Is the queryFn Not Called?

**Common Issue**: `initialData` + `staleTime` combination

**Problem**:

```typescript
// 🚨 queryFn won't be called on mount
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  initialData: [],
  staleTime: 5 * 1000,  // Data is "fresh" for 5 seconds
})
```

**Why**: `initialData` is cached and considered "fresh" for `staleTime` duration, so no background refetch.

**Best Practice**: Use `placeholderData` for fallback values:

```typescript
// ✅ Always triggers background refetch
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  placeholderData: [],  // ✅ Not cached, always refetches
  staleTime: 5 * 1000,
})
```

**Alternative**: Mark `initialData` as stale:

```typescript
// ✅ Triggers background update
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  initialData: [],
  initialDataUpdatedAt: 0,  // ✅ Mark as stale
  staleTime: 5 * 1000,
})
```

**For dynamic query keys**: Be specific about which query gets `initialData`:

```typescript
// ✅ Only page 0 gets initialData
const [page, setPage] = React.useState(0)

const { data } = useQuery({
  queryKey: ['todos', page],
  queryFn: () => fetchTodos(page),
  initialData: page === 0 ? initialDataForPageZero : undefined,
  staleTime: 5 * 1000,
})
```

**Why**: Each `queryKey` is a separate cache entry. `initialData` applies to all keys unless you conditionally provide it.

## Not To Do

### 1. Don't Try to Pass Parameters to refetch

```typescript
// 🚨 This doesn't work
const { refetch } = useQuery({
  queryKey: ['item'],
  queryFn: () => fetchItem({ id: 1 }),
})

<button onClick={() => refetch({ id: 2 })}>Show Item 2</button>
```

**Why**: Different parameters should be different cache entries. This would overwrite cache.

### 2. Don't Create QueryClient Inside Component Without Stabilizing

```typescript
// 🚨 Cache lost on re-render
export default function App() {
  const queryClient = new QueryClient()  // ❌ New client every render
  
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

### 3. Don't Import QueryClient Directly

```typescript
// 🚨 Loses decoupling benefits
export const queryClient = new QueryClient()

// In another file
import { queryClient } from './queryClient'  // ❌ Tight coupling
```

### 4. Don't Use Fetch API Without Error Checking

```typescript
// 🚨 4xx/5xx won't be treated as errors
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    const response = await fetch('/todos/' + todoId)
    return response.json()  // ❌ No error check
  },
})
```

### 5. Don't Catch Errors Without Re-throwing

```typescript
// 🚨 Returns successful Promise<void>
useQuery({
  queryKey: ['todos', todoId],
  queryFn: async () => {
    try {
      const { data } = await axios.get('/todos/' + todoId)
      return data
    } catch (error) {
      console.error(error)
      // ❌ No re-throw - query succeeds!
    }
  },
})
```

### 6. Don't Use initialData for Fallback Values

```typescript
// 🚨 Prevents background refetch
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  initialData: [],  // ❌ Empty array is fallback, not real data
  staleTime: 5 * 1000,
})
```

## Key Insights

1. **Parameters = QueryKey**: Put all variables in `queryKey`, not passed to `refetch`
2. **Declarative > Imperative**: Change dependencies, React Query handles the rest
3. **Stable QueryClient**: Create outside component or use `useState` for stability
4. **useQueryClient hook**: Always use hook, not direct import
5. **Fetch API quirks**: Check `response.ok` and throw on errors
6. **Re-throw errors**: Always re-throw after logging
7. **initialData vs placeholderData**: Use `placeholderData` for fallbacks, `initialData` for real cached data
