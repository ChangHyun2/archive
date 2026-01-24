---
description: "React Query FAQs: parameters in queryKey not refetch, ensure QueryClient is stable, use useQueryClient hook, handle fetch API errors"
alwaysApply: false
---

# React Query FAQs Rules

## How to Pass Parameters to refetch?

- **You cannot pass parameters to `refetch`**
  - Parameters are dependencies to your query - they should be in the `queryKey`
  - Use declarative approach - put parameters in `queryKey`

```typescript
// ✅ Declarative approach
const [id, setId] = useState(1)

const { data } = useQuery({
  queryKey: ['item', id],  // ✅ id is in queryKey
  queryFn: () => fetchItem({ id }),
})

<button onClick={() => setId(2)}>Show Item 2</button>
```

- **Better**: Use URL state for shareable links

```typescript
const { id } = useParams()

const { data } = useQuery({
  queryKey: ['item', id],
  queryFn: () => fetchItem({ id }),
})
```

- **For smooth transitions**: Use `keepPreviousData`

```typescript
import { keepPreviousData } from '@tanstack/react-query'

const { data, isPlaceholderData } = useQuery({
  queryKey: ['item', id],
  queryFn: () => fetchItem({ id }),
  placeholderData: keepPreviousData,  // ✅ Shows previous data while loading
})
```

## Why Are Updates Not Shown?

### Query Keys Not Matching

- **Ensure exact key matching (type matters)**

```typescript
// 🚨 These don't match!
['item', '1']  // string
['item', 1]    // number

// ✅ Use consistent types
const id = Number(useParams().id)  // Convert if needed
queryKey: ['item', id]
```

- **Solution**: Use TypeScript and Query Key Factories to prevent this.

### QueryClient Not Stable

- **Create `QueryClient` outside component or use stable reference**

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

// ✅ If you must create inside App - use useState
export default function App() {
  const [queryClient] = React.useState(() => new QueryClient())
  
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  )
}
```

- **Why**: New `QueryClient` = new empty cache. Unstable client means cache is lost on re-renders.

## Use useQueryClient() Hook

- **Always use `useQueryClient()` hook instead of importing**

```typescript
// ✅ Use hook
const queryClient = useQueryClient()
```

- **Reasons:**
  1. Consistency: `useQuery` uses `useQueryClient` internally - same client everywhere
  2. Decoupling: Works with any client in context (useful for testing with different defaults)
  3. SSR/Microfrontends: Client might be created inside App (can't export)
  4. Hooks in defaults: If you need other hooks in `QueryClient` defaults, must create inside App

## Handle fetch API Errors

- **React Query needs a rejected Promise for error handling**
  - Libraries like `axios` provide this automatically
  - `fetch` API requires manual transformation

```typescript
const queryFn = async () => {
  const response = await fetch('/api/todos')
  if (!response.ok) {
    throw new Error('Failed to fetch')  // ✅ Transform to rejected Promise
  }
  return response.json()
}
```

## placeholderData vs initialData

- **Use `initialData` for pre-filling from cache**
  - Persisted to cache, respects `staleTime`
- **Use `placeholderData` for fake data while loading**
  - Never persisted, always refetches

## Don't Do

- **Don't use `refetch` to pass parameters**
  - `refetch` is for refetching with the same parameters
  - Fix: Put parameters in `queryKey` instead

- **Don't create unstable `QueryClient`**
  - Cache is lost on re-renders
  - Fix: Create outside component or use `useState`

- **Don't import `QueryClient` directly**
  - Doesn't work with SSR/microfrontends
  - Fix: Use `useQueryClient()` hook

- **Don't forget to transform `fetch` API errors**
  - React Query needs a rejected Promise
  - Fix: Check `response.ok` and throw error if not ok
