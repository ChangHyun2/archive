---
description: "React Query as state manager: customize staleTime instead of turning off refetches, understand stale-while-revalidate, use setQueryDefaults"
alwaysApply: false
---

# React Query as State Manager Rules

## React Query is an Async State Manager

- **React Query is NOT a data fetching library - it's an Async State Manager**
  - It doesn't fetch data for you (you use fetch, axios, etc.)
  - It manages any form of asynchronous state (anything that returns a Promise)
  - It's a proper, real "global state manager"

## Customize staleTime Instead of Turning Off Refetches

- **Don't turn off refetch flags** unless you know it makes sense. Instead, **customize `staleTime`**

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // ✅ Globally default to 20 seconds
      staleTime: 1000 * 20,
    },
  },
})

// ✅ Everything todo-related will have 1 minute staleTime
queryClient.setQueryDefaults(
  todoKeys.all,
  { staleTime: 1000 * 60 }
)
```

- **Key insight**: As long as data is **fresh**, it will always come from cache only. No network request, no matter how often you retrieve it.

## Understand Stale-While-Revalidate

- **React Query uses stale-while-revalidate caching:**
  - Shows cached (potentially stale) data immediately
  - Performs background refetch to revalidate
  - **Principle**: Stale data is better than no data (no loading spinner = feels faster)

## Use setQueryDefaults for Granular Control

- **Set defaults per Query Key granularity**

```typescript
// Everything todo-related gets 1 minute staleTime
queryClient.setQueryDefaults(
  todoKeys.all,
  { staleTime: 1000 * 60 }
)
```

- This follows the same partial matching as Query Filters, so you can set defaults at any level of your key hierarchy.

## Don't Do

- **Don't turn off refetch flags unnecessarily**
  - Don't turn off `refetchOnMount` or `refetchOnWindowFocus` just because you see "too many" requests
  - Customize `staleTime` instead

- **Don't pass data as props to avoid refetches**
  - While passing data as props is fine and explicit, don't do it just to avoid React Query refetches
  - You'll lose the benefit of background updates when components mount later

- **Don't use React Query for client state**
  - React Query is for **async/server state**
  - Use dedicated solutions (zustand, xstate/store) for client state

- **Don't sync server data to another state manager**
  - You lose all background updates
  - You duplicate state management logic
  - You bypass React Query's smart refetching
