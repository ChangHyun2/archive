# Seeding the Query Cache - Best Practices

## Key Concept

**Fetch waterfalls** occur when one request waits for another to complete. Seeding the cache early prevents waterfalls, especially with Suspense.

## The Problem: Fetch Waterfalls

### What Are Waterfalls?

A waterfall describes a situation where one request is made, and we wait for it to complete before firing another request.

**Dependent queries** (unavoidable):
- First request contains info needed for second request
- Must wait for first to complete

**Independent queries** (should be parallel):
- Can fetch all data in parallel
- Waterfalls are unnecessary and slow

### Suspense Waterfalls

**Problem**: When using multiple `useSuspenseQuery` in the same component:

1. Component renders, tries to read first query
2. No data in cache → suspends
3. Component unmounts, fallback renders
4. First fetch finishes, component remounts
5. First query read successfully
6. Component sees second query, tries to read it
7. No data in cache → suspends (again!)
8. Second query fetched
9. Component finally renders

**Result**: Fallback shows much longer than necessary.

## Best Practices

### 1. Prefetch Queries Before Component Renders

**Best Practice**: Use `prefetchQuery` to initiate fetches early:

```typescript
const issuesQuery = { queryKey: ['issues'], queryFn: fetchIssues }

// ✅ Initiate fetch before component renders
queryClient.prefetchQuery(issuesQuery)

function Issues() {
  const issues = useSuspenseQuery(issuesQuery)  // ✅ Data already fetching
}
```

**Why**: 
- Fetch starts as soon as JavaScript bundle is evaluated
- Works well with route-based code splitting
- Component renders with data already in cache or fetching

**For multiple queries**: Prefetch all of them:

```typescript
// ✅ Prefetch both in parallel
queryClient.prefetchQuery(issuesQuery)
queryClient.prefetchQuery(labelsQuery)

function Component() {
  const issues = useSuspenseQuery(issuesQuery)
  const labels = useSuspenseQuery(labelsQuery)  // ✅ Both already fetching
}
```

### 2. Use `useSuspenseQueries` (v5+)

**Best Practice**: Use dedicated hook for parallel suspense queries:

```typescript
// ✅ v5+ - triggers all fetches in parallel
const [issues, labels] = useSuspenseQueries({
  queries: [
    { queryKey: ['issues'], queryFn: fetchIssues },
    { queryKey: ['labels'], queryFn: fetchLabels },
  ],
})
```

**Why**: Automatically triggers all fetches in parallel, avoiding waterfalls.

### 3. Seed Detail Cache from List Cache (Pull Approach)

**Best Practice**: Use `initialData` to pull data from list cache:

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
  })
}
```

**Benefits**:
- Seeds cache "just in time"
- No extra network request if data exists in list
- Works when navigating from list to detail

**Important**: Add `initialDataUpdatedAt` to handle staleness:

```typescript
const useTodo = (id: number) => {
  const queryClient = useQueryClient()
  return useQuery({
    queryKey: ['todos', 'detail', id],
    queryFn: () => fetchTodo(id),
    initialData: () => {
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

**Why**: Without `initialDataUpdatedAt`, React Query uses `Date.now()`, making data appear fresh. With it, staleness is measured from when list was fetched.

### 4. Seed Detail Cache When Fetching List (Push Approach)

**Best Practice**: Create detail caches when fetching list:

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

**Benefits**:
- `staleTime` automatically respected (measured from list fetch)
- Detail data ready immediately when navigating

**Drawbacks**:
- Might create unnecessary cache entries (if detail never visited)
- Pushed data might be garbage collected (inactive queries)
- No good callback to hook into (must do in `queryFn`)

### 5. Fetch on Server or in Router Loaders

**Best Practice**: Fetch as early as possible:

- **Server Side Rendering**: Fetch on server
- **Router loaders**: Prefetch in route loaders (React Router, Remix)
- **Route-based code splitting**: Prefetch when route code loads

**Key principle**: The earlier you initiate a fetch, the sooner it can finish.

### 6. Stick to One Query Per Component (When Using Suspense)

**Best Practice**: Avoid multiple `useSuspenseQuery` in same component:

```typescript
// ✅ One query per component
function Issues() {
  const { data } = useSuspenseQuery({
    queryKey: ['issues'],
    queryFn: fetchIssues,
  })
  return <IssuesList data={data} />
}

function Labels() {
  const { data } = useSuspenseQuery({
    queryKey: ['labels'],
    queryFn: fetchLabels,
  })
  return <LabelsList data={data} />
}

// ✅ Use both in parent
function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Issues />
      <Labels />
    </Suspense>
  )
}
```

**Why**: Each component can suspend independently, but prefetching both avoids waterfalls.

## Not To Do

### 1. Don't Use Multiple Suspense Queries Without Prefetching

```typescript
// 🚨 Creates waterfall
function Component() {
  const issues = useSuspenseQuery({ queryKey: ['issues'], queryFn: fetchIssues })
  const labels = useSuspenseQuery({ queryKey: ['labels'], queryFn: fetchLabels })
  // ❌ Second query waits for first to complete
}
```

**Fix**: Prefetch both, or use `useSuspenseQueries`.

### 2. Don't Forget `initialDataUpdatedAt` When Seeding

```typescript
// 🚨 Data appears fresh even if list is stale
const useTodo = (id: number) => {
  const queryClient = useQueryClient()
  return useQuery({
    queryKey: ['todos', 'detail', id],
    queryFn: () => fetchTodo(id),
    initialData: () => {
      return queryClient.getQueryData(['todos', 'list'])?.find(...)
    },
    // ❌ Missing initialDataUpdatedAt
    staleTime: 5000,  // Won't refetch if list was fetched 20 minutes ago
  })
}
```

**Fix**: Add `initialDataUpdatedAt` to respect list's staleness.

### 3. Don't Use Push Approach for Large Lists

```typescript
// 🚨 Creates many unnecessary cache entries
const useTodos = () => {
  const queryClient = useQueryClient()
  return useQuery({
    queryKey: ['todos', 'list'],
    queryFn: async () => {
      const todos = await fetchTodos()  // 1000 items
      todos.forEach((todo) => {
        queryClient.setQueryData(['todos', 'detail', todo.id], todo)
        // ❌ Creates 1000 cache entries, most never used
      })
      return todos
    },
  })
}
```

**Fix**: Use pull approach for large lists, or only push for likely-to-be-visited items.

### 4. Don't Seed If Data Structures Don't Match

```typescript
// 🚨 List item missing required field
const useTodo = (id: number) => {
  return useQuery({
    queryKey: ['todos', 'detail', id],
    queryFn: () => fetchTodo(id),  // Returns { id, title, description, comments }
    initialData: () => {
      // ❌ List item only has { id, title } - missing description, comments
      return queryClient.getQueryData(['todos', 'list'])?.find(...)
    },
  })
}
```

**Fix**: Use `placeholderData` instead if structures don't match (see #9).

## Key Insights

1. **Waterfalls are slow**: Sequential fetches take longer than parallel
2. **Suspense amplifies waterfalls**: Each query suspends separately
3. **Prefetch early**: Start fetches before components render
4. **useSuspenseQueries**: v5+ hook for parallel suspense queries
5. **Pull approach**: Seed from list cache when needed (just-in-time)
6. **Push approach**: Seed when fetching list (proactive, but can create unused entries)
7. **initialDataUpdatedAt**: Essential for correct staleness when seeding
8. **One query per component**: Easier to manage with Suspense
