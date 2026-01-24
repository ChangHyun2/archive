---
description: "When React Query might not be needed: new apps with Server Components, frameworks with built-in data fetching. When it still is: hybrid approach, client-only apps"
alwaysApply: false
---

# You Might Not Need React Query Rules

## When You Might Not Need React Query

### New Applications with Server Components

- **If you're starting a new app with Server Components (Next.js App Router, Remix), you might not need React Query**

```typescript
// ✅ Server Component - no React Query needed
export default async function Page() {
  const data = await fetch(
    `https://api.github.com/repos/tanstack/react-query`
  )

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.description}</p>
    </div>
  )
}
```

- **Why**: 
  - Server Components fetch data on server
  - No client-side async state management needed
  - Framework handles data fetching
  - React Query is primarily for **client-side** async state

### Framework Has Built-in Data Fetching

- **Use framework's built-in solutions:**
  - **Next.js**: Server Components, Server Actions
  - **Remix**: Loaders, Actions
  - **SvelteKit**: Load functions

- **Key principle**: If your framework has first-class support for data fetching, use it. Don't add React Query just because.

### Static Data with staleTime: Infinity

- **Move static data fetching to Server Components**

```typescript
// ✅ If you had this with React Query
useQuery({
  queryKey: ['static-data'],
  queryFn: fetchStaticData,
  staleTime: Infinity,  // Never refetches
})

// Consider Server Component instead
export default async function Page() {
  const data = await fetchStaticData()
  return <div>{/* render */}</div>
}
```

## When You Still Need React Query

### Hybrid Approach with Server Components

- **Combine Server Components with React Query for client-side features**

```typescript
// Server Component - prefetch first page
export default async function Page() {
  const firstPage = await fetchPage(0)
  
  return <InfiniteList initialData={firstPage} />
}

// Client Component - fetch more pages on scroll
'use client'
function InfiniteList({ initialData }) {
  const { data, fetchNextPage } = useInfiniteQuery({
    queryKey: ['items'],
    queryFn: fetchPage,
    initialData: { pages: [initialData], pageParams: [0] },
  })
  
  // Infinite scroll logic
}
```

- **Use cases:**
  - Infinite scrolling (prefetch on server, load more on client)
  - Offline support
  - Background refetching (interval fetching, auto-refetch on focus)
  - Optimistic updates
  - Real-time updates

### Client-Only Applications

- **If your app is client-only (SPA), React Query is still valuable**
  - Caching
  - Background refetching
  - Request deduplication
  - Global state management

## Best Practices

- **Evaluate your use case**
  - Do you need client-side async state management?
  - Does your framework provide first-class data fetching?
  - Do you need advanced features (offline, optimistic updates, real-time)?

- **Incremental adoption**
  - You can use React Query alongside Server Components
  - Use Server Components for initial data
  - Use React Query for client-side features

- **Use the right tool for the job**
  - Server Components for server-side data fetching
  - React Query for client-side async state management
  - Don't force React Query where it's not needed
