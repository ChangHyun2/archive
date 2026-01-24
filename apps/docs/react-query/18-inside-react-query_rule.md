---
description: "React Query architecture: keep QueryClient stable, understand QueryCache, Query, QueryObserver, active vs inactive queries"
alwaysApply: false
---

# React Query Architecture Rules

## Keep QueryClient Stable

- **Create `QueryClient` once and make it stable**

```typescript
// ✅ Created once, stable reference
const queryClient = new QueryClient()

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <RestOfYourApp />
    </QueryClientProvider>
  )
}
```

- **Key Points**:
  - `QueryClient` is a container for `QueryCache` and `MutationCache`
  - Distributed via React Context (no re-renders, just access)
  - Provides convenience methods to work with caches
  - Holds default options for queries and mutations

## Understand QueryCache

- **Key Points**:
  - In-memory object (not persisted by default)
  - Keys: `queryKeyHash` (stably serialized `queryKey`)
  - Values: `Query` class instances
  - **Important**: Cache is lost on page reload (use persisters for localStorage)

## Understand Query and QueryObserver

- **Query**:
  - Contains all query information (data, status, meta)
  - Executes `queryFn`
  - Handles retry, cancellation, de-duplication
  - Has internal state machine (prevents impossible states)
  - Knows which Observers are interested
  - Can inform Observers about changes

- **QueryObserver**:
  - Created when you call `useQuery`
  - Subscribed to exactly one Query
  - **Where optimizations happen**:
    - Tracks which Query properties component uses
    - Only notifies about relevant changes
    - Handles `select` option for fine-grained subscriptions
    - Manages timers (`staleTime`, interval fetching)

## Active vs Inactive Queries

- **Active**: Has at least one Observer
- **Inactive**: No Observers, but still in cache (greyed out in DevTools)
- Number on left in DevTools = number of Observers

## Don't Do

- **Don't create unstable `QueryClient`**
  - New `QueryClient` = new empty cache
  - Fix: Create once outside component or use `useState` for stability
