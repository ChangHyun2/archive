# How Infinite Queries work - Best Practices

## Key Concepts

### Infinite Queries vs Single Queries

**Infinite queries** are React Query's way to implement pagination/doom-scrolling pages. They're identical to single queries in most ways:

- Same Query class in cache
- Same retryer logic
- Same deduplication
- **Difference**: Data structure and how data is retrieved

### QueryBehavior Pattern

Infinite queries use a **QueryBehavior** pattern. The queryFn is wrapped by `infiniteQueryBehavior`:

```typescript
class Query() {
  fetch() {
    if (this.state.fetchStatus === 'idle') {
      this.#dispatch({ type: 'fetch' })
      this.#retryer = createRetryer(
        fetchFn: this.options.behavior.onFetch(  // ✅ Behavior wraps queryFn
          this.context,
          this.options.queryFn
        ),
        retry: this.options.retry,
        retryDelay: this.options.retryDelay
      )
      return this.#retryer.start()
    }

    return this.#retryer.promise
  }
}
```

**Conceptually brilliant**: To make a query infinite, just attach `infiniteQueryBehavior` - no changes to caching, revalidation, or subscriptions.

## Best Practices

### 1. Understand Retry Behavior

**Important**: `retry: 3` means **three retries over all pages**, not three retries per page.

This is consistent with single queries - it's three retries per query, no matter how often it actually fetches.

### 2. Use maxPages to Limit Cache Size

If you have many pages in cache (e.g., user scrolled down 100 pages), use `maxPages` to limit cache size:

```typescript
useInfiniteQuery({
  queryKey: ['items'],
  queryFn: fetchPage,
  maxPages: 10,  // ✅ Only keep last 10 pages in cache
})
```

**Why**: 
- Prevents too many pages in cache
- Speeds up rendering when navigating
- Reduces memory usage

### 3. Understand How Pages Are Structured

Infinite queries structure data as a linked list:
- Each page depends on the previous one
- Pages are stored in `{ pages: [...] }` format
- `getNextPageParam` determines the next page parameter

## Not To Do

### 1. Don't Expect Per-Page Retries

Retries apply to the entire query operation, not individual pages. If page 3 fails during a refetch, the retry will restart from page 1.

### 2. Don't Use Same QueryKey for useQuery and useInfiniteQuery

```typescript
// 🚨 This won't work - they share the same cache entry
useQuery({
  queryKey: ['items'],
  queryFn: fetchItems,
})

useInfiniteQuery({
  queryKey: ['items'],  // ❌ Same key!
  queryFn: fetchPage,
})

// ✅ Use different keys
useInfiniteQuery({
  queryKey: ['infiniteItems'],  // ✅ Different key
  queryFn: fetchPage,
})
```

**Why**: They have fundamentally different data structures and would conflict.

## Key Insights

1. **Infinite queries use QueryBehavior**: Attaching behavior is all that's needed - no architectural changes
2. **Retries restart the loop**: If a page in the middle fails, retry restarts from the beginning (fixed in recent versions with closure hoisting)
3. **Use maxPages**: Limit cache size for better performance
4. **Different keys required**: Can't share QueryKey with regular queries
