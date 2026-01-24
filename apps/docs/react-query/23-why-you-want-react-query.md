# Why You Want React Query - Best Practices

## Key Concept

**Data fetching is simple. Async state management is not.** React Query is NOT a data fetching library - it's an **async state manager**.

## Common Bugs in useEffect Data Fetching

### The "Simple" useEffect Example

```typescript
// 🚨 This code has 5 bugs!
function Bookmarks({ category }) {
  const [data, setData] = useState([])
  const [error, setError] = useState()

  useEffect(() => {
    fetch(`${endpoint}/${category}`)
      .then(res => res.json())
      .then(d => setData(d))
      .catch(e => setError(e))
  }, [category])
}
```

## Best Practices (How React Query Fixes These)

### 1. Fix Race Conditions

**Bug**: Network responses can arrive out of order. If you change category from "books" to "movies" and the movies response arrives before books, you'll show wrong data.

**Best Practice**: React Query stores state by `queryKey` (input), preventing race conditions:

```typescript
// ✅ No race condition - state stored by queryKey
function Bookmarks({ category }) {
  const { isLoading, data, error } = useQuery({
    queryKey: ['bookmarks', category],  // ✅ Different key = different cache entry
    queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
  })
}
```

**Why**: Each `queryKey` is a separate cache entry. Changing category = different key = separate state.

### 2. Provide Loading State

**Bug**: No way to show pending UI while requests are happening.

**Best Practice**: React Query provides loading state automatically:

```typescript
// ✅ Loading state included
const { isLoading, data, error } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})

if (isLoading) return <SkeletonLoader />
```

**Why**: Loading, data, and error states are provided for free, including discriminated unions on type level.

### 3. Distinguish Empty vs No Data

**Bug**: Initializing `data` with `[]` makes it impossible to distinguish "no data yet" from "no data at all".

**Best Practice**: React Query separates these states:

```typescript
// ✅ Clear separation
const { isLoading, data, error } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})

// isLoading = true → no data yet
// isLoading = false && data = [] → no data at all
// isLoading = false && data = [...] → has data
```

**Why**: Can further enhance with `placeholderData` for better UX.

### 4. Reset State on Parameter Change

**Bug**: `data` and `error` don't reset when `category` changes. You can end up with:
- `data`: data from current category
- `error`: error from previous category

**Best Practice**: React Query automatically resets state per `queryKey`:

```typescript
// ✅ Each queryKey has its own state
const { data, error } = useQuery({
  queryKey: ['bookmarks', category],  // ✅ Different category = clean state
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})
```

**Why**: Each `queryKey` is independent. Changing category = new query = fresh state.

### 5. Handle StrictMode Double Fetches

**Bug**: React StrictMode calls effects twice in development. Without proper cleanup, this causes issues.

**Best Practice**: React Query deduplicates fetches automatically:

```typescript
// ✅ Deduplication handles StrictMode
const { data } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: () => fetch(`${endpoint}/${category}`).then(res => res.json()),
})
```

**Why**: Multiple calls with same `queryKey` = one fetch, shared promise.

### 6. Handle HTTP Errors

**Best Practice**: Check `response.ok` and throw:

```typescript
// ✅ Proper error handling
const { data, error } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: () =>
    fetch(`${endpoint}/${category}`).then((res) => {
      if (!res.ok) {
        throw new Error('Failed to fetch')
      }
      return res.json()
    }),
})
```

**Why**: `fetch` doesn't reject on 4xx/5xx - must check `res.ok` manually.

### 7. Request Cancellation

**Best Practice**: Use `signal` for automatic cancellation:

```typescript
// ✅ Automatic cancellation
const { data, error } = useQuery({
  queryKey: ['bookmarks', category],
  queryFn: ({ signal }) =>
    fetch(`${endpoint}/${category}`, { signal }).then((res) => {
      if (!res.ok) {
        throw new Error('Failed to fetch')
      }
      return res.json()
    }),
})
```

**Why**: When `category` changes, previous request is automatically aborted.

## Not To Do

### 1. Don't Use useEffect Without Cleanup

```typescript
// 🚨 Race condition bug
useEffect(() => {
  fetch(`${endpoint}/${category}`)
    .then(res => res.json())
    .then(d => setData(d))  // ❌ Can set wrong data if response arrives late
    .catch(e => setError(e))
}, [category])
```

**Fix**: Add cleanup with ignore flag, or use React Query.

### 2. Don't Initialize Data with Empty Array

```typescript
// 🚨 Can't distinguish "loading" from "empty"
const [data, setData] = useState([])  // ❌ Is this loading or actually empty?

// ✅ Initialize with undefined
const [data, setData] = useState()
```

### 3. Don't Forget to Reset State

```typescript
// 🚨 Old error persists when category changes
useEffect(() => {
  fetch(`${endpoint}/${category}`)
    .then(res => res.json())
    .then(d => setData(d))  // ❌ Doesn't reset error
    .catch(e => setError(e))  // ❌ Doesn't reset data
}, [category])
```

**Fix**: Manually reset both, or use React Query.

### 4. Don't Ignore StrictMode

```typescript
// 🚨 Fires twice in StrictMode
useEffect(() => {
  fetch(`${endpoint}/${category}`)  // ❌ Called twice
    .then(res => res.json())
    .then(d => setData(d))
}, [category])
```

**Fix**: Add cleanup, or use React Query (handles automatically).

### 5. Don't Forget HTTP Error Handling

```typescript
// 🚨 4xx/5xx won't be treated as errors
fetch(`${endpoint}/${category}`)
  .then(res => res.json())  // ❌ No error check
  .then(d => setData(d))
```

**Fix**: Check `res.ok` and throw, or use React Query.

## The React Query Solution

**Simple, bug-free code**:

```typescript
// ✅ All bugs fixed automatically
function Bookmarks({ category }) {
  const { isLoading, data, error } = useQuery({
    queryKey: ['bookmarks', category],
    queryFn: ({ signal }) =>
      fetch(`${endpoint}/${category}`, { signal }).then((res) => {
        if (!res.ok) {
          throw new Error('Failed to fetch')
        }
        return res.json()
      }),
  })

  // Return JSX based on data and error state
}
```

**Benefits**:
- ✅ No race conditions (state stored by `queryKey`)
- ✅ Loading state included
- ✅ Clear empty state separation
- ✅ State resets automatically on parameter change
- ✅ Deduplication handles StrictMode
- ✅ Request cancellation built-in

## Key Insights

1. **Data fetching ≠ Async state management**: Fetching is easy, managing state is hard
2. **5 bugs in 10 lines**: Simple `useEffect` has race conditions, missing loading, state issues, StrictMode problems
3. **React Query fixes all automatically**: Same amount of code, but bug-free
4. **queryKey is the key**: Different keys = different cache entries = no race conditions
5. **Automatic deduplication**: Multiple calls = one fetch
6. **Request cancellation**: Built-in with `signal`
7. **Type-safe states**: Discriminated unions for loading/data/error
