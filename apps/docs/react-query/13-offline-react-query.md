# Offline React Query - Best Practices

## Key Concept

React Query handles offline scenarios well with caching, but v4 introduced `networkMode` to better control when queries should run based on network connection.

## Understanding Query States

### Status vs fetchStatus

**Two separate state dimensions**:

- **`status`**: Information about **data**
  - `success`: You have data
  - `loading`: You don't have data yet
  - `error`: Query failed

- **`fetchStatus`**: Information about **queryFn** execution
  - `fetching`: Query is really executing (request in-flight)
  - `paused`: Query is not executing (paused until connection regained)
  - `idle`: Query is currently not running

**Key insight**: You can be in `success` state and `paused` (data exists, but background refetch paused), or `loading` state and `paused` (query mounted but can't fetch).

## Best Practices

### 1. Use `networkMode: 'online'` (Default, v4+)

**Best Practice**: Use default `online` mode for most queries:

```typescript
// ✅ Default behavior (v4+)
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  networkMode: 'online',  // ✅ Default - requires network
})
```

**Behavior**:
- Query only runs if network connection is active
- If offline, query goes to `paused` state
- No network request fired when offline
- Prevents unnecessary failed requests

**Use when**: Most data fetching queries that require network.

### 2. Use `networkMode: 'offlineFirst'` for Browser Cache/Service Workers

**Best Practice**: Use when you have additional caching layers:

```typescript
// ✅ For browser cache or service workers
useQuery({
  queryKey: ['repo'],
  queryFn: fetchRepo,
  networkMode: 'offlineFirst',  // ✅ First request always fires
})
```

**Behavior**:
- First request **always** fires (even when offline)
- If cache hit → success
- If cache miss → network error → retries paused → `paused` state
- Works with browser cache headers (e.g., `cache-control: max-age=60`)
- Works with service workers for offline-first PWAs

**Why**: To intercept a fetch request (for caching), it must happen. If React Query doesn't fire the request, browser cache/service worker can't intercept it.

**Example**: GitHub API sends `cache-control: public, max-age=60` - browser cache works offline for 60 seconds.

### 3. Use `networkMode: 'always'` for Non-Network Queries

**Best Practice**: Use for queries that don't need network:

```typescript
// ✅ For web workers, expensive async processing
useQuery({
  queryKey: ['processed-data'],
  queryFn: async () => {
    // Expensive processing in web worker
    return processDataInWorker()
  },
  networkMode: 'always',  // ✅ Never paused
})
```

**Behavior**:
- Queries always fire, never paused
- Network connection doesn't matter
- Useful for non-network async operations

**Use when**: 
- Web workers
- Expensive async processing
- Any query that doesn't actually need network

### 4. React to `fetchStatus` for Better UX

**Best Practice**: Use `isPaused` to show appropriate UI:

```typescript
function Issues() {
  const { data, isPaused, fetchStatus } = useQuery({
    queryKey: ['issues'],
    queryFn: fetchIssues,
  })

  if (isPaused) {
    return <OfflineIndicator />  // ✅ Show offline state
  }

  if (data) {
    return <IssuesList data={data} />
  }

  return <Loading />
}
```

**Why**: Better UX - users know why data isn't loading (offline vs loading).

## Not To Do

### 1. Don't Use `online` Mode with Browser Cache

```typescript
// 🚨 Browser cache won't work
useQuery({
  queryKey: ['repo'],
  queryFn: fetchRepo,
  networkMode: 'online',  // ❌ Won't fire request when offline
  // Browser cache can't intercept if request doesn't fire
})
```

**Problem**: If React Query doesn't fire the request, browser cache/service worker can't intercept it.

**Fix**: Use `networkMode: 'offlineFirst'` if you rely on browser cache.

### 2. Don't Assume Queries Need Network

```typescript
// 🚨 Query doesn't need network but gets paused
useQuery({
  queryKey: ['processed-data'],
  queryFn: async () => {
    // Web worker processing - no network needed
    return processInWorker()
  },
  // ❌ Default 'online' mode pauses this unnecessarily
})
```

**Problem**: Query that doesn't need network gets paused when offline.

**Fix**: Use `networkMode: 'always'` for non-network queries.

### 3. Don't Ignore `fetchStatus` When Offline Support Matters

```typescript
// 🚨 No way to show offline state
function Issues() {
  const { data, isLoading } = useQuery({
    queryKey: ['issues'],
    queryFn: fetchIssues,
  })

  if (isLoading) {
    return <Loading />  // ❌ Shows loading even when paused/offline
  }

  return <IssuesList data={data} />
}
```

**Problem**: Can't distinguish between "loading" and "paused/offline".

**Fix**: Check `isPaused` or `fetchStatus` to show appropriate UI.

### 4. Don't Use `offlineFirst` Without Additional Cache Layer

```typescript
// 🚨 Unnecessary network request when offline
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  networkMode: 'offlineFirst',  // ❌ Fires request even when offline
  // But no browser cache/service worker to intercept
})
```

**Problem**: Fires request when offline, but no cache to serve it.

**Fix**: Only use `offlineFirst` if you have browser cache or service workers.

## Key Insights

1. **Two state dimensions**: `status` (data) vs `fetchStatus` (execution)
2. **networkMode controls behavior**: `online` (default), `offlineFirst`, `always`
3. **online mode**: Requires network, pauses when offline (v4+ default)
4. **offlineFirst mode**: Always fires first request, useful for browser cache/service workers
5. **always mode**: Never pauses, for non-network queries
6. **paused state**: New in v4, visible in DevTools
7. **isPaused flag**: Derived from `fetchStatus`, use for offline UI
8. **Backward compatible**: Can ignore `fetchStatus`, works like v3
