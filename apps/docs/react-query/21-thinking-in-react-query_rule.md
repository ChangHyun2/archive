---
description: "React Query mindset: it's an async state manager not data fetching library, staleTime is your best friend, treat parameters as dependencies"
alwaysApply: false
---

# Thinking in React Query Rules

## React Query is NOT a Data Fetching Library

- **React Query is an Async State Manager, not a data fetching library**
  - It doesn't fetch data for you (you use fetch, axios, etc.)
  - It only cares if you return a fulfilled or rejected Promise
  - That's it!

```typescript
// ✅ React Query needs queryKey and queryFn
const { data } = useQuery({
  queryKey: ['issues'],
  queryFn: () => axios.get('/api/issues').then(res => res.data),
  //     ^^^^ THIS is your data fetching library (axios, fetch, etc.)
})
```

- **What React Query doesn't care about:**
  - How you fetch (axios, fetch, graphql-request, etc.)
  - baseURL configuration
  - Response headers
  - GraphQL vs REST

- **Questions that disappear:**
  - "How can I define a baseURL with React Query?" → Use your fetch library
  - "How can I access response headers?" → Use your fetch library
  - "How can I make GraphQL requests?" → Use your fetch library

## staleTime is Your Best Friend

- **Set `staleTime` based on your needs**

```typescript
// ✅ Config that only changes on server restart
useQuery({
  queryKey: ['config'],
  queryFn: fetchConfig,
  staleTime: Infinity,  // ✅ Never goes stale
})

// ✅ Highly collaborative tool
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  staleTime: 0,  // ✅ Always refetch (default)
})

// ✅ Set global default, override when needed
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,  // ✅ 5 minutes default
    },
  },
})
```

- **How it works:**
  - **Fresh data**: Given from cache only, no refetch
  - **Stale data**: Given from cache AND background refetch
- **Important**: `staleTime` defaults to **zero** (0ms), so React Query marks everything as stale instantly.

## Treat Parameters as Dependencies

- **Put any variable used in `queryFn` into the `queryKey`**

```typescript
// ✅ Filters in queryKey
const useIssues = (filters: Filters) => {
  return useQuery({
    queryKey: ['issues', filters],  // ✅ Filters in key
    queryFn: () => fetchIssues(filters),  // ✅ Used in queryFn
  })
}
```

- **Why this is critical:**
  1. Separate cache entries: Different filters = different keys = separate cache
  2. Avoids race conditions: Each filter combination cached separately
  3. Automatic refetches: Changing filters = new key = automatic fetch
  4. Avoids stale closures: No closure issues when filters change

- **Use ESLint plugin to enforce this**

## Separate Server and Client State

- **Separate server state (React Query) from client state (other managers)**

```typescript
// ✅ Server state managed by React Query
const useIssues = () => {
  const filters = useStore(state => state.filters)  // ✅ Client state from Zustand
  return useQuery({
    queryKey: ['issues', filters],  // ✅ Server state
    queryFn: () => fetchIssues(filters),
  })
}
```

- **Why**: Clear separation of concerns. React Query doesn't care where filters come from. Use URL, Zustand, Redux, local state - whatever fits.

## Don't Do

- **Don't think React Query fetches data**
  - React Query doesn't care about data fetching configuration
  - Fix: Use your fetch library for baseURL, headers, etc.

- **Don't turn off all refetches**
  - React Query refetches for good reasons (data can be outdated)
  - Fix: Set appropriate `staleTime` instead

- **Don't forget parameters in `queryKey`**
  - Race conditions and stale closures
  - Fix: Put everything used in `queryFn` into `queryKey`

- **Don't sync state from React Query**
  - React Query is already a state manager
  - Fix: Use React Query directly, don't copy its state elsewhere

- **Don't use `onSuccess` for state syncing**
  - Deprecated and anti-pattern
  - Fix: Use data directly from `useQuery`
