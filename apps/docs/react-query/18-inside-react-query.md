# Inside React Query - Architecture Overview

## Key Architecture Concepts

### 1. QueryClient

**Best Practice**: Create `QueryClient` once and make it stable:

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

**Key Points**:
- `QueryClient` is a container for `QueryCache` and `MutationCache`
- Distributed via React Context (no re-renders, just access)
- Provides convenience methods to work with caches
- Holds default options for queries and mutations

### 2. QueryCache

**Key Points**:
- In-memory object (not persisted by default)
- Keys: `queryKeyHash` (stably serialized `queryKey`)
- Values: `Query` class instances
- **Important**: Cache is lost on page reload (use persisters for localStorage)

### 3. Query

**Key Points**:
- Contains all query information (data, status, meta)
- Executes `queryFn`
- Handles retry, cancellation, de-duplication
- Has internal state machine (prevents impossible states)
- Knows which Observers are interested
- Can inform Observers about changes

**State Machine Benefits**:
- Prevents impossible states (e.g., can't be loading and success)
- De-duplicates fetches (if already fetching, reuses promise)
- Handles cancellation (returns to previous state)

### 4. QueryObserver

**Key Points**:
- Created when you call `useQuery`
- Subscribed to exactly one Query
- **Where optimizations happen**:
  - Tracks which Query properties component uses
  - Only notifies about relevant changes
  - Handles `select` option for fine-grained subscriptions
  - Manages timers (`staleTime`, interval fetching)

**Example**: If component only uses `data`, it won't re-render when `isFetching` changes.

### 5. Active vs Inactive Queries

**Key Points**:
- **Active**: Has at least one Observer
- **Inactive**: No Observers, but still in cache (greyed out in DevTools)
- Number on left in DevTools = number of Observers

## Component Flow

**Typical flow when component mounts**:

1. Component mounts → calls `useQuery`
2. `useQuery` creates `QueryObserver`
3. Observer subscribes to `Query` in `QueryCache`
4. Subscription might:
   - Create new `Query` (if doesn't exist)
   - Trigger background refetch (if data is stale)
5. Query state changes → Observer informed
6. Observer runs optimizations → decides if component should update
7. Component re-renders with new state
8. Query finishes → Observer informed again

**Ideal flow**: Data already in cache when component mounts (see #17: Seeding the Query Cache).

## Framework-Agnostic Design

**Key Insight**: Most logic lives in framework-agnostic Query Core:
- `QueryClient`
- `QueryCache`
- `Query`
- `QueryObserver`

**Why this matters**:
- Easy to create adapters for new frameworks
- React adapter: ~100 lines of code
- Solid adapter: ~100 lines of code
- Logic happens outside React/Solid/Vue
- Updates propagated to Observer → Observer decides if component updates

## Best Practices

### 1. Keep QueryClient Stable

```typescript
// ✅ Stable reference
const queryClient = new QueryClient()

// ❌ Don't recreate on every render
function App() {
  const queryClient = new QueryClient()  // New cache every render!
}
```

### 2. Understand Observer-Level Optimizations

- Each `useQuery` call creates a `QueryObserver`
- Observer tracks which properties you use
- Only notifies about relevant changes
- Use `select` for fine-grained subscriptions

### 3. Cache is In-Memory Only

- Default: Cache lost on page reload
- Use persisters if you need persistence
- Each `QueryKey` = separate cache entry

### 4. Query State Machine Prevents Issues

- De-duplication: Multiple calls = one fetch
- Cancellation: Returns to previous state
- Impossible states prevented

## Key Insights

1. **QueryClient is a container**: Holds caches and defaults, provides convenience methods
2. **QueryCache is in-memory**: Keys are `queryKeyHash`, values are `Query` instances
3. **Query has the logic**: State machine, retry, cancellation, de-duplication
4. **Observer is the optimizer**: Tracks usage, decides when to notify component
5. **Framework-agnostic core**: Most logic outside React, adapters are thin
6. **Active vs Inactive**: Queries without Observers are inactive but still cached
7. **Component flow**: Mount → Observer → Query → State changes → Optimizations → Render
