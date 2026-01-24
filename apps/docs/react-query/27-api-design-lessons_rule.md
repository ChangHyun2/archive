---
description: "React Query API design principles: evolve with complexity, design with TypeScript, use inversion of control, take time before adding features"
alwaysApply: false
---

# React Query API Design Rules

## API Should Evolve with App Complexity

- **React Query's API is designed to be minimal and intuitive for simple use cases, powerful and flexible for complex scenarios**

**Starting point** (80% of value with minimal options):
```typescript
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
})
```

This simple API gives you:
- Caching
- Request deduplication
- Stale-while-revalidate background updates
- Global state management
- Automatic garbage collection
- Loading/error states + retries

**As complexity grows**:
- Add `useMutation` for updates
- Add optimistic updates when needed
- Add infinite queries when needed
- Use advanced features (persisters, direct cache subscriptions) only when necessary

## Design APIs with TypeScript in Mind

- **Think about types from the beginning**
  - "If something is hard for a compiler to figure out, it's also hard for humans to understand"
  - Since v5, `useQuery` only accepts options syntax (reduced type code by 80%)

## Use Inversion of Control for Flexibility

- **Instead of adding every requested feature, make options accept callback functions**

**Example - Debouncing**:
```typescript
// ❌ Don't add debounce option to React Query
// ✅ Use user-land solution
const [filter, setFilter] = useState('')
const debouncedFilter = useDeferredValue(filter)

useQuery({
  queryKey: ['todos', debouncedFilter],
  queryFn: () => fetchTodos(debouncedFilter),
})
```

**Example - Conditional refetchOnWindowFocus**:
```typescript
// ✅ Make option accept a function
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  refetchOnWindowFocus: (query) => {
    // Don't refetch if query is in error state
    return query.state.status !== 'error'
  },
})
```

- **Benefit**: Keeps API surface small while giving users flexibility to implement features themselves.

## Take Time Before Adding Features

- **Don't rush to add requested features**
  - Will this work for everybody?
  - What about cases the requester hasn't considered?
  - Once added, we can't change it without a major release

- **Better solution**: Solve the real problem, not the symptom
  - Example: `maxPages` option that limits cache size holistically, rather than `refetchPage` API

## Major Versions Are About Breaking Changes, Not Features

- **Important**: Major versions are not about features - they're about breaking existing APIs
  - Features mostly go in minors
  - Breaking changes require major versions

## Don't Do

- **Don't add features outside responsibility**
  - React Query is for async state management
  - Don't add debouncing, throttling, etc. - use user-land solutions

- **Don't rush decisions**
  - Take time to understand the problem fully
  - Consider all use cases before adding features

- **Don't have multiple ways for the same thing**
  - Keep API consistent
  - Avoid confusion and maintenance burden

- **Don't ship without beta feedback**
  - Get feedback from community before releasing
  - Major versions can't be changed easily
