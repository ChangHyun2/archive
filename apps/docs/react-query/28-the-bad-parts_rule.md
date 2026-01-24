---
description: "React Query limitations: no normalized caching, not for client state. Learn incrementally, use the right tool for the job"
alwaysApply: false
---

# React Query Limitations Rules

## No Normalized Caching

- **React Query uses a document cache (complete response stored under key), not normalized caching**

**What this means**:
- If a task is both `status: open` AND `priority: high`, it will be in both caches
- Data duplication can occur
- No automatic relationship management

**Why**: React Query only knows Promises - it doesn't know what's inside the cache. Normalized caching requires schema awareness and relation knowledge (like GraphQL clients have).

**When it matters**: If you're using GraphQL and need normalized caching, React Query might not be the right choice. Consider Apollo Client, urql, or the community tool `normy`.

**Tradeoff**: For most applications, refetching upon invalidation works well and is easier to understand.

## Don't Use for Client State

- **Don't use React Query for synchronous client state** (like sidebar toggle)

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

// ✅ Use dedicated solution
const [isOpen, setIsOpen] = useState(false)
// or
const isOpen = useStore(state => state.isOpen)
```

- **Why**: React Query is for **async/server state**. Use dedicated solutions (zustand, xstate/store) for client state.

## Learning Curve

- **React Query's API is designed to evolve with you**

**Start simple** (80% of value):
- `useQuery` with minimal options gives you: caching, request deduplication, stale-while-revalidate, background updates, global state management, automatic garbage collection, loading/error states, retries

**Add complexity gradually**:
- Add `useMutation` for updates
- Add optimistic updates when needed
- Add infinite queries when needed
- Use advanced features (persisters, direct cache subscriptions) only when necessary

You don't need to learn everything from the start.

## Best Practices

- **Learn incrementally**
  - Start with basic `useQuery`
  - Add features as you need them
  - Don't try to learn everything at once

- **Use the right tool for the job**
  - React Query for async/server state
  - Other tools for client state
  - Framework solutions for server-side data fetching
