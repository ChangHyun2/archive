---
description: "React Query WebSockets: use query invalidation for events, set high staleTime, clean up connections, use setQueriesData for partial updates"
alwaysApply: false
---

# React Query WebSockets Rules

## Setup Queries as Usual

- **Setup queries normally, as if not using WebSockets**
  - Most of the time, you still have HTTP endpoints for initial data fetching
  - WebSockets are for real-time updates

## Use Query Invalidation for Event-Based Updates

- **Send events from backend, invalidate queries on WebSocket messages**

```typescript
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient()
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/')
    websocket.onmessage = (event) => {
      const data = JSON.parse(event.data)
      const queryKey = [...data.entity, data.id].filter(Boolean)
      queryClient.invalidateQueries({ queryKey })  // ✅ Invalidate on event
    }

    return () => {
      websocket.close()
    }
  }, [queryClient])
}
```

- **Event format examples:**
  - `{ "entity": ["posts", "list"] }` → Invalidates posts list
  - `{ "entity": ["posts", "detail"], "id": 5 }` → Invalidates single post
  - `{ "entity": ["posts"] }` → Invalidates everything post-related

## Set High staleTime with WebSockets

- **Use `staleTime: Infinity` when updating via WebSockets**

```typescript
// ✅ Global default for WebSocket apps
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: Infinity,  // ✅ No automatic refetches
    },
  },
})
```

- **Why**: WebSockets update data in real-time, no need for automatic refetches

## Clean Up WebSocket Connection

- **Always close WebSocket in cleanup**

```typescript
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient()
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/')
    
    return () => {
      websocket.close()  // ✅ Cleanup
    }
  }, [queryClient])
}
```

## Use `setQueriesData` for Partial Updates

- **For frequent, small updates, update cache directly**

```typescript
websocket.onmessage = (event) => {
  const data = JSON.parse(event.data)
  queryClient.setQueriesData(data.entity, (oldData) => {
    const update = (entity) =>
      entity.id === data.id
        ? { ...entity, ...data.payload }  // ✅ Partial update
        : entity
    return Array.isArray(oldData)
      ? oldData.map(update)
      : update(oldData)
  })
}
```

## Don't Do

- **Don't push complete data objects**
  - Pushes data even when not needed
  - Wastes bandwidth
  - Fix: Use event-based invalidation or partial updates

- **Don't forget to clean up WebSocket**
  - Memory leak
  - Fix: Always return cleanup function

- **Don't use automatic refetches with WebSockets**
  - WebSockets already update data, automatic refetches are redundant
  - Fix: Set `staleTime: Infinity` or high value

- **Don't use `setQueryData` for multiple query keys**
  - Only updates one query key
  - Fix: Use `setQueriesData` for multiple keys, or prefer invalidation
