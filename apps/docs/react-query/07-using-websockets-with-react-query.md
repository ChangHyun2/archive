# Using WebSockets with React Query - Best Practices

## Key Concept

React Query works well with WebSockets. Since React Query only needs a Promise (resolved or rejected), it doesn't care how data arrives - HTTP or WebSocket.

## Best Practices

### 1. Setup Queries as Usual

**Best Practice**: Setup queries normally, as if not using WebSockets:

```typescript
// ✅ Standard queries
const usePosts = () =>
  useQuery({ queryKey: ['posts', 'list'], queryFn: fetchPosts })

const usePost = (id) =>
  useQuery({
    queryKey: ['posts', 'detail', id],
    queryFn: () => fetchPost(id),
  })
```

**Why**: Most of the time, you still have HTTP endpoints for initial data fetching. WebSockets are for real-time updates.

### 2. Use Query Invalidation for Event-Based Updates

**Best Practice**: Send events from backend, invalidate queries on WebSocket messages:

```typescript
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient()
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/')
    websocket.onopen = () => {
      console.log('connected')
    }
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

**Event format examples**:
- `{ "entity": ["posts", "list"] }` → Invalidates posts list
- `{ "entity": ["posts", "detail"], "id": 5 }` → Invalidates single post
- `{ "entity": ["posts"] }` → Invalidates everything post-related

**Benefits**:
- Avoids over-pushing (only refetches if observer is active)
- Works with fuzzy matching
- Simple and effective
- If not on Posts page, won't refetch until you navigate there

### 3. Use `setQueriesData` for Partial Updates

**Best Practice**: For frequent, small updates, update cache directly:

```typescript
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient()
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/')
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

    return () => {
      websocket.close()
    }
  }, [queryClient])
}
```

**Use when**:
- Big datasets with small, frequent updates
- Only specific fields change (e.g., title, like count)
- Want to avoid full refetch

**Tradeoffs**:
- More complex
- Doesn't handle additions/deletions
- TypeScript might not like it
- Consider invalidation instead

### 4. Set High staleTime with WebSockets

**Best Practice**: Use `staleTime: Infinity` when updating via WebSockets:

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

**Why**: 
- WebSockets update data in real-time
- No need for automatic refetches (window focus, mount, etc.)
- Data fetched initially via `useQuery`, then from cache
- Refetching only happens via explicit invalidation

**Alternative**: Set per-query if only some queries use WebSockets.

### 5. Clean Up WebSocket Connection

**Best Practice**: Always close WebSocket in cleanup:

```typescript
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient()
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/')
    
    // Setup handlers...
    
    return () => {
      websocket.close()  // ✅ Cleanup
    }
  }, [queryClient])
}
```

**Why**: Prevents memory leaks and unnecessary connections.

## Not To Do

### 1. Don't Push Complete Data Objects

```typescript
// 🚨 Over-pushing complete data
websocket.onmessage = (event) => {
  const data = JSON.parse(event.data)
  queryClient.setQueryData(['posts', data.id], data.post)  // ❌ Complete object
}
```

**Problem**: 
- Pushes data even when not needed
- Wastes bandwidth
- More complex than invalidation

**Fix**: Use event-based invalidation or partial updates.

### 2. Don't Forget to Clean Up WebSocket

```typescript
// 🚨 Memory leak
React.useEffect(() => {
  const websocket = new WebSocket('wss://echo.websocket.org/')
  // ❌ No cleanup
}, [])
```

**Fix**: Always return cleanup function.

### 3. Don't Use Automatic Refetches with WebSockets

```typescript
// 🚨 Unnecessary refetches
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 0,  // ❌ Will refetch on focus, mount, etc.
    },
  },
})
```

**Problem**: WebSockets already update data, automatic refetches are redundant.

**Fix**: Set `staleTime: Infinity` or high value.

### 4. Don't Use `setQueryData` for Multiple Query Keys

```typescript
// 🚨 Only updates one query key
websocket.onmessage = (event) => {
  const data = JSON.parse(event.data)
  queryClient.setQueryData(['posts', 'detail', data.id], updatedPost)
  // ❌ Doesn't update list view
}
```

**Problem**: If you have list and detail views, need to update both.

**Fix**: Use `setQueriesData` for multiple keys, or prefer invalidation.

## Key Insights

1. **React Query is agnostic**: Works with any Promise source (HTTP, WebSocket, etc.)
2. **Event-based invalidation**: Send events, invalidate queries - simple and effective
3. **Avoids over-pushing**: Only refetches if observer is active
4. **Partial updates**: Use `setQueriesData` for frequent small updates
5. **High staleTime**: Set `staleTime: Infinity` when using WebSockets
6. **Cleanup**: Always close WebSocket connection
7. **Initial fetch via HTTP**: WebSockets for updates, HTTP for initial data
8. **Fuzzy matching**: Works great with query invalidation patterns
