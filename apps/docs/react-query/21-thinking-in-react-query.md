# Thinking in React Query - Best Practices

## Key Concept

**3 simple mindset shifts** to approach React Query correctly, similar to tying your shoes correctly - a small tweak makes a huge difference.

## The 3 Core Principles

### 1. React Query is NOT a Data Fetching Library

**Key Insight**: React Query is an **Async State Manager**, not a data fetching library.

**Best Practice**: Understand what React Query does and doesn't do:

```typescript
// ✅ React Query needs queryKey and queryFn
const { data } = useQuery({
  queryKey: ['issues'],
  queryFn: () => axios.get('/api/issues').then(res => res.data),
  //     ^^^^ THIS is your data fetching library (axios, fetch, etc.)
})
```

**What React Query cares about**:
- Returns a fulfilled or rejected Promise
- That's it!

**What React Query doesn't care about**:
- How you fetch (axios, fetch, graphql-request, etc.)
- baseURL configuration
- Response headers
- GraphQL vs REST

**Best Practice**: Use any data fetching library you want:

```typescript
// ✅ All of these work - React Query doesn't care
useQuery({
  queryKey: ['data'],
  queryFn: () => axios.get('/api/data').then(res => res.data),
})

useQuery({
  queryKey: ['data'],
  queryFn: () => fetch('/api/data').then(res => res.json()),
})

useQuery({
  queryKey: ['data'],
  queryFn: () => graphqlRequest(query),
})
```

**Questions that disappear**:
- "How can I define a baseURL with React Query?" → Use your fetch library
- "How can I access response headers?" → Use your fetch library
- "How can I make GraphQL requests?" → Use your fetch library

**Answer**: React Query doesn't care! Just return a Promise.

### 2. staleTime is Your Best Friend

**Key Insight**: React Query is a data synchronization tool, but it won't blindly refetch everything.

**Best Practice**: Set `staleTime` based on your needs:

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

**How it works**:
- **Fresh data**: Given from cache only, no refetch
- **Stale data**: Given from cache AND background refetch

**Important**: `staleTime` defaults to **zero** (0ms), so React Query marks everything as stale instantly. This is aggressive but errs on the side of keeping things up-to-date.

**Best Practice**: Define `staleTime` based on your resource:
- Static config: `staleTime: Infinity`
- Collaborative data: `staleTime: 0`
- Most cases: Set a reasonable default (e.g., 5 minutes)

### 3. Treat Parameters as Dependencies

**Key Insight**: Put any variable used in `queryFn` into the `queryKey`.

**Best Practice**: Add all parameters to `queryKey`:

```typescript
// ✅ Filters in queryKey
const useIssues = (filters: Filters) => {
  return useQuery({
    queryKey: ['issues', filters],  // ✅ Filters in key
    queryFn: () => fetchIssues(filters),  // ✅ Used in queryFn
  })
}
```

**Why this is critical**:
1. **Separate cache entries**: Different filters = different keys = separate cache
2. **Avoids race conditions**: Each filter combination cached separately
3. **Automatic refetches**: Changing filters = new key = automatic fetch
4. **Avoids stale closures**: No closure issues when filters change

**Best Practice**: Use ESLint plugin to enforce this:

```typescript
// ✅ ESLint plugin will catch this
useQuery({
  queryKey: ['issues'],  // ❌ Missing filters!
  queryFn: () => fetchIssues(filters),  // Uses filters but not in key
})
```

**Think of it like**: `queryKey` is like `useEffect` dependency array, but without referential stability concerns. No need for `useMemo` or `useCallback`.

### 4. Use QueryClient for Client State

**Best Practice**: Separate server state (React Query) from client state (other managers):

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

**Why**: 
- Clear separation of concerns
- React Query doesn't care where filters come from
- Use URL, Zustand, Redux, local state - whatever fits
- Query automatically runs when filters change

## Not To Do

### 1. Don't Think React Query Fetches Data

```typescript
// 🚨 Wrong mindset
"How do I configure baseURL in React Query?"
"How do I add headers in React Query?"
"How do I make GraphQL requests with React Query?"

// ✅ Correct mindset
"React Query doesn't care - use your fetch library"
```

### 2. Don't Turn Off All Refetches

```typescript
// 🚨 Over-reacting to refetches
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  refetchOnMount: false,
  refetchOnWindowFocus: false,
  refetchOnReconnect: false,
  // ❌ Turning off everything
})

// ✅ Better: Set appropriate staleTime
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  staleTime: 1000 * 60 * 5,  // ✅ 5 minutes
})
```

**Why**: React Query refetches for good reasons (data can be outdated). Use `staleTime` to control when data is considered fresh.

### 3. Don't Forget Parameters in queryKey

```typescript
// 🚨 Race conditions and stale closures
const useIssues = (filters: Filters) => {
  return useQuery({
    queryKey: ['issues'],  // ❌ Missing filters
    queryFn: () => fetchIssues(filters),  // Uses filters
  })
}

// ✅ Correct
const useIssues = (filters: Filters) => {
  return useQuery({
    queryKey: ['issues', filters],  // ✅ Filters in key
    queryFn: () => fetchIssues(filters),
  })
}
```

### 4. Don't Sync State from React Query

```typescript
// 🚨 Anti-pattern: State syncing
useEffect(() => {
  if (query.data) {
    setLocalData(query.data)  // ❌ Unnecessary
  }
}, [query.data])

// ✅ Use React Query directly
const { data } = useQuery({ ... })
// data is already available, no need to sync
```

**Why**: React Query is already a state manager. Don't put its state into another one.

### 5. Don't Use onSuccess for State Syncing

```typescript
// 🚨 Deprecated and anti-pattern
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  onSuccess: (data) => {
    setLocalState(data)  // ❌ State syncing
  },
})

// ✅ Use data directly
const { data } = useQuery({ ... })
```

## Key Insights

1. **React Query = Async State Manager**: Not a data fetching library
2. **staleTime controls refetching**: Set it based on your needs, not zero everywhere
3. **Parameters = Dependencies**: Put everything used in `queryFn` into `queryKey`
4. **Use ESLint plugin**: Enforces parameter-in-key rule automatically
5. **No referential stability needed**: Unlike `useEffect`, no `useMemo`/`useCallback` needed
6. **Separate server/client state**: React Query for server state, other tools for client state
7. **Call useQuery anywhere**: It's a state manager - use it wherever you need the data
8. **Don't sync state**: React Query is already managing state, don't copy it elsewhere
