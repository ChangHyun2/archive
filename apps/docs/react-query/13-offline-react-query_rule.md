---
description: "React Query offline: use networkMode (online/offlineFirst/always), react to fetchStatus, understand status vs fetchStatus"
alwaysApply: false
---

# React Query Offline Rules

## Understanding Query States

- **Two separate state dimensions:**
  - **`status`**: Information about **data** (`success`, `loading`, `error`)
  - **`fetchStatus`**: Information about **queryFn** execution (`fetching`, `paused`, `idle`)
- **Key insight**: You can be in `success` state and `paused` (data exists, but background refetch paused), or `loading` state and `paused` (query mounted but can't fetch)

## Use networkMode: 'online' (Default, v4+)

- **Use default `online` mode for most queries**
  - Query only runs if network connection is active
  - If offline, query goes to `paused` state
  - No network request fired when offline
  - Prevents unnecessary failed requests

```typescript
// ✅ Default behavior (v4+)
useQuery({
  queryKey: ['issues'],
  queryFn: fetchIssues,
  networkMode: 'online',  // ✅ Default - requires network
})
```

## Use networkMode: 'offlineFirst' for Browser Cache/Service Workers

- **Use when you have additional caching layers**
  - First request **always** fires (even when offline)
  - If cache hit → success
  - If cache miss → network error → retries paused → `paused` state
  - Works with browser cache headers (e.g., `cache-control: max-age=60`)
  - Works with service workers for offline-first PWAs

```typescript
// ✅ For browser cache or service workers
useQuery({
  queryKey: ['repo'],
  queryFn: fetchRepo,
  networkMode: 'offlineFirst',  // ✅ First request always fires
})
```

- **Why**: To intercept a fetch request (for caching), it must happen. If React Query doesn't fire the request, browser cache/service worker can't intercept it.

## Use networkMode: 'always' for Non-Network Queries

- **Use for queries that don't need network**
  - Queries always fire, never paused
  - Network connection doesn't matter
  - Useful for non-network async operations (web workers, expensive processing)

```typescript
// ✅ For web workers, expensive async processing
useQuery({
  queryKey: ['processed-data'],
  queryFn: async () => {
    return processDataInWorker()
  },
  networkMode: 'always',  // ✅ Never paused
})
```

## React to fetchStatus for Better UX

- **Use `isPaused` to show appropriate UI**

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

## Don't Do

- **Don't use `online` mode with browser cache**
  - Browser cache won't work if React Query doesn't fire the request
  - Fix: Use `networkMode: 'offlineFirst'` if you rely on browser cache

- **Don't assume queries need network**
  - Query that doesn't need network gets paused when offline
  - Fix: Use `networkMode: 'always'` for non-network queries

- **Don't ignore `fetchStatus` when offline support matters**
  - Can't distinguish between "loading" and "paused/offline"
  - Fix: Check `isPaused` or `fetchStatus` to show appropriate UI

- **Don't use `offlineFirst` without additional cache layer**
  - Fires request when offline, but no cache to serve it
  - Fix: Only use `offlineFirst` if you have browser cache or service workers
