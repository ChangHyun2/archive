# React Query API Design - Lessons Learned

## Key Principles

### 1. API Should Evolve with App Complexity

React Query's API is designed to be:
- **Minimal and intuitive** for simple use cases
- **Powerful and flexible** for complex scenarios

**The scale**: As app complexity grows, APIs should become more powerful & flexible.

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

### 2. Design APIs with TypeScript in Mind

**Best Practice**: Think about types from the beginning. Don't "just make it work" first and figure out types later.

**Key insight**: "If something is hard for a compiler to figure out, it's also hard for humans to understand."

**Example**: `useQuery` used to accept 3 different positional argument patterns. This required:
- Multiple overloads (80% more type code)
- Runtime checks to transform different versions
- Poor error messages

**Solution**: Since v5, `useQuery` only accepts options syntax. This reduced type code by 80% (from 125 to 25 lines).

### 3. Use Inversion of Control for Flexibility

Instead of adding every requested feature, make options accept **callback functions** to allow users to implement custom behavior.

**Example 1 - Debouncing**:
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

**Example 2 - Conditional refetchOnWindowFocus**:
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

**Benefit**: Keeps API surface small while giving users flexibility to implement features themselves.

### 4. Take Time Before Adding Features

**Best Practice**: Don't rush to add requested features. Users are demanding, but maintainers need the bigger picture.

**Questions to ask**:
- Will this work for everybody?
- What about cases the requester hasn't considered?
- Once added, we can't change it without a major release

**Example of a mistake**: `refetchPage` API for infinite queries
- Added too quickly without fully understanding the problem
- API was weird and confusing
- Only worked for imperative methods, not automatic refetches
- Could cause correctness issues

**Better solution**: `maxPages` option that limits cache size holistically, solving the real problem (too many pages in cache) rather than the symptom (wanting to refetch one page).

### 5. Major Versions Are About Breaking Changes, Not Features

**Important**: Major versions are not about features - they're about breaking existing APIs. Features mostly go in minors.

**Examples**:
- React Hooks: 16.8 (minor)
- React Router loaders: 6.4 (minor)
- Bun Windows support: 1.1 (minor)

**Problem**: Users expect "new features" in major versions, but majors are about breaking changes. This creates pressure to add features just for marketing.

**Lesson**: When a new version comes out, don't think "what's new" - ask "what's breaking" instead.

## Not To Do

### 1. Don't Add Features That Aren't Your Responsibility

**Example**: Debouncing API calls
- Not React Query's responsibility
- Many ways to implement debouncing
- Adds bundle size
- Can be implemented in user-land easily

**Solution**: Use inversion of control - make options accept functions so users can implement features themselves.

### 2. Don't Rush API Decisions

Take time to understand the **real problem** being solved, not just the immediate request. The first solution might not be the best.

### 3. Don't Use Multiple Ways to Achieve the Same Thing

**Example**: `useQuery` used to accept 3 different argument patterns. This:
- Increased type complexity (overloads)
- Required runtime checks
- Confused users

**Solution**: One clear API (options syntax only).

### 4. Don't Ship Without Beta Feedback

**Critical**: Try out beta versions and report feedback. This is the best time to be heard.

**Problem**: Users expect maintainers to get everything right, but willingness to try betas and report feedback is limited.

**Result**: Mistakes make it into "stable" releases, and "stable" just means "we can't change it anymore" - not "bug-free" or "battle-tested".

**Call to action**: Help maintainers by trying betas and reporting feedback. Open source is a two-way street.

## Lessons for Library Maintainers

1. **API design is hard** - especially in open source where you can't easily revert decisions
2. **Think about types from the beginning** - don't add dynamic constructs that are hard to type
3. **Use inversion of control** - make options accept functions for flexibility
4. **Take your time** - understand the real problem before adding features
5. **Major versions are about breaking changes** - features go in minors
6. **Beta feedback is crucial** - encourage users to try betas and report issues
