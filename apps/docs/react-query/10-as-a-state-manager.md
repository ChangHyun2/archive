# React Query as a State Manager - Best Practices

## Key Insight

**React Query is NOT a data fetching library** - it's an **Async State Manager**.

- It doesn't fetch data for you (you use fetch, axios, etc.)
- It manages any form of asynchronous state (anything that returns a Promise)
- It's a proper, real "global state manager"

## Best Practices

### 1. Let React Query Manage Global Async State

React Query provides global state management through QueryKeys. Same key = same data, anywhere in your app:

```typescript
export const useTodos = () =>
  useQuery({ queryKey: ['todos'], queryFn: fetchTodos })

function ComponentOne() {
  const { data } = useTodos()
}

function ComponentTwo() {
  // ✅ Will get exactly the same data as ComponentOne
  const { data } = useTodos()
}
```

**Benefits**:
- Components can be anywhere in the tree
- React Query deduplicates requests (only one network request even if multiple components call it)
- No prop drilling needed

### 2. Understand Stale-While-Revalidate

React Query uses **stale-while-revalidate** caching:
- Shows cached (potentially stale) data immediately
- Performs background refetch to revalidate
- **Principle**: Stale data is better than no data (no loading spinner = feels faster)

### 3. Customize staleTime Instead of Turning Off Refetches

**Don't turn off refetch flags** unless you know it makes sense. Instead, **customize `staleTime`**:

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // ✅ Globally default to 20 seconds
      staleTime: 1000 * 20,
    },
  },
})

// ✅ Everything todo-related will have 1 minute staleTime
queryClient.setQueryDefaults(
  todoKeys.all,
  { staleTime: 1000 * 60 }
)
```

**Key insight**: As long as data is **fresh**, it will always come from cache only. No network request, no matter how often you retrieve it.

**Why this matters**: If you conditionally mount components, you might see multiple refetches:

```typescript
// ⚠️ ComponentTwo mounts conditionally, triggers second request
function ComponentOne() {
  const { data } = useTodos()
  if (data) {
    return <ComponentTwo />  // Mounts after data exists
  }
  return <Loading />
}

function ComponentTwo() {
  const { data } = useTodos()  // ⚠️ Triggers refetch
}
```

**Solution**: Set `staleTime` so data stays fresh for your use case. Then ComponentTwo will get cached data.

### 4. Understand Smart Refetches

React Query triggers refetches at strategic points:

- **refetchOnMount**: When a new component calling useQuery mounts
- **refetchOnWindowFocus**: When you focus the browser tab (great for production - user comes back)
- **refetchOnReconnect**: When network reconnects
- **Manual**: `queryClient.invalidateQueries()`

**Note**: `refetchOnWindowFocus` might seem excessive in development (switching tabs often), but in production it's perfect - user comes back from checking email/Twitter and sees latest updates.

### 5. Use setQueryDefaults for Granular Control

Set defaults per Query Key granularity:

```typescript
// Everything todo-related gets 1 minute staleTime
queryClient.setQueryDefaults(
  todoKeys.all,
  { staleTime: 1000 * 60 }
)
```

This follows the same partial matching as Query Filters, so you can set defaults at any level of your key hierarchy.

### 6. Don't Sync Server Data to Another State Manager

**Resist the urge** to sync server data to Redux/Context/etc. Let React Query manage it.

**Why**:
- You lose all background updates
- You duplicate state management logic
- You bypass React Query's smart refetching

## Not To Do

### 1. Don't Turn Off Refetch Flags Unnecessarily

Don't turn off `refetchOnMount` or `refetchOnWindowFocus` just because you see "too many" requests. Customize `staleTime` instead.

### 2. Don't Pass Data as Props to Avoid Refetches

While passing data as props is fine and explicit, don't do it just to avoid React Query refetches. You'll lose the benefit of background updates when components mount later.

**Example**: If ComponentTwo mounts after user clicks a button (minutes later), a background refetch ensures fresh data. This wouldn't work if you bypassed React Query.

### 3. Don't Use React Query for Client State

React Query is for **async/server state**. Use dedicated solutions (zustand, xstate/store) for client state.

### 4. Don't Worry About "Mixing Responsibilities"

It's fine to use `useQuery` in components at any layer. The old "smart vs dumb" component pattern led to prop drilling and boilerplate. Hooks allow dependency injection anywhere.

**Tradeoff**: Component is more coupled to React Query, but also more independent (can move it anywhere and it works).

## Key Takeaways

1. **React Query = Async State Manager**, not data fetching library
2. **Stale-while-revalidate**: Show stale data immediately, refetch in background
3. **Customize staleTime**: Better than turning off refetch flags
4. **Let React Query do its magic**: Don't sync to other state managers
5. **Smart refetches**: Strategic points ensure data stays fresh
6. **Global state**: Same QueryKey = same data everywhere
