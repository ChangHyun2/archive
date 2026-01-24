# React Query - The Bad Parts (Tradeoffs)

## Myths Debunked

### 1. Bundle Size is Huge

**Myth**: React Query has a huge bundle size.

**Reality**: 
- **Not what you see on npm** (700kb+ includes sources, source-maps, codemods)
- **Not what you see on bundlephobia** (uses legacy build for older bundlers)
- **Actual size**: ~9.63 kB minzipped for core features (QueryClient, QueryClientProvider, useQuery, useMutation)
- **Full library**: ~12.4 kB minzipped (when using every feature)

**Key insight**: Bundle size you **save** by code you don't have to write. React Query "pays for itself" because the more you use it, the more it saves you code that you would otherwise have to write yourself. Most custom solutions would likely be larger or fail in edge cases.

### 2. Can't Fetch on Button Click

**Myth**: React Query can't do imperative data fetching.

**Reality**: React Query is **declarative by default**, but this is by design.

```typescript
// ❌ This doesn't work - refetch doesn't accept arguments
const { refetch } = useQuery({
  queryKey: ['tasks'],
  queryFn: fetchTasks,
})
refetch(filters)  // Won't work!

// ✅ Declarative approach - put state in query key
const [filters, setFilters] = useState()
const { data } = useQuery({
  queryKey: ['tasks', filters],  // State in key
  queryFn: () => fetchTasks(filters),
})
```

**Why**: If we had a static key and refetched with different arguments, we would:
- Overwrite previously cached data
- Run into race conditions (like fetching in useEffect)

React Query solves both problems by making dependencies part of the QueryKey.

**Alternative**: With TanStack Router, use search params instead of React state:
```typescript
// Type-safe, sharable URLs, browser back button support
const { filters } = useSearch()
const { data } = useQuery({
  queryKey: ['tasks', filters],
  queryFn: () => fetchTasks(filters),
})
```

### 3. Learning Curve is Too Steep

**Reality**: React Query's API is designed to evolve with you.

**Start simple** (80% of value):
- `useQuery` with minimal options gives you: caching, request deduplication, stale-while-revalidate, background updates, global state management, automatic garbage collection, loading/error states, retries

**Add complexity gradually**:
- Add `useMutation` for updates
- Add optimistic updates when needed
- Add infinite queries when needed
- Use advanced features (persisters, direct cache subscriptions) only when necessary

You don't need to learn everything from the start.

## Actual Limitations

### 1. No Normalized Caching

React Query uses a **document cache** (complete response stored under key), not normalized caching.

**What this means**:
- If a task is both `status: open` AND `priority: high`, it will be in both caches
- Data duplication can occur
- No automatic relationship management

**Why**: React Query only knows Promises - it doesn't know what's inside the cache. Normalized caching requires schema awareness and relation knowledge (like GraphQL clients have).

**When it matters**: If you're using GraphQL and need normalized caching, React Query might not be the right choice. Consider Apollo Client, urql, or the community tool `normy`.

**Tradeoff**: For most applications, refetching upon invalidation works well and is easier to understand.

### 2. Don't Use for Client State

**Don't use React Query for synchronous client state** (like sidebar toggle).

```typescript
// ❌ Don't do this
const { data: isOpen } = useQuery({
  queryKey: ['sidebarState'],
  queryFn: () => {},  // No async work!
  initialData: false,
  staleTime: Infinity,
  refetchOnWindowFocus: false,
  // ... many more configs to disable
})
```

**Why it's bad**:
- Verbose and inefficient
- Not easy to get right
- React Query is designed for async state management

**Use the right tool**:
- **zustand**: Minimal, efficient, un-opinionated
- **xstate/store**: Better TypeScript support, event-driven
- **React Context + useState**: For simple cases

**Key principle**: Client state and server state have different needs. Use the right tool for the right job.

### 3. Why Not Built into React?

**Question**: Why isn't data fetching built into React?

**Answer**: React team has a bigger vision:
- **Suspense**: Decouples components from loading/error states, works great with TypeScript
- **Server Components**: Extends React to the server
- **React Compiler**: Will eventually optimize data access automatically

**Current state**: Suspense and Server Components ARE the async primitive we've been waiting for. If you can use a framework that supports Server Components, use them. Until then, use React Query.

## Best Practices (From Tradeoffs)

### 1. Think Declaratively, Not Imperatively

Instead of "If I click this button, I want to refetch", think "I want data that matches this state". How it changes is irrelevant.

### 2. Use Query Keys for Dependencies

Put any variable used in `queryFn` into the `queryKey`. This makes queries declarative and prevents race conditions.

### 3. Learn Incrementally

Start with `useQuery` and `useMutation`. Add complexity only when needed.

### 4. Use Right Tool for Right Job

- **React Query**: Async server state
- **zustand/xstate**: Client state
- **React Context**: Simple shared state

### 5. Consider Normalized Caching Needs

If you need normalized caching (especially with GraphQL), consider alternatives or use `normy`.

## Not To Do

### 1. Don't Worry About Bundle Size (Within Reason)

9-12kb is reasonable for what React Query provides. The code it saves you is likely more.

### 2. Don't Try to Make React Query Imperative

Use declarative approach with query keys. If you need imperative fetching, React Query might not be the right tool.

### 3. Don't Use React Query for Client State

Use dedicated client state management libraries instead.

### 4. Don't Try to Learn Everything at Once

Start simple, add complexity gradually as your app grows.
