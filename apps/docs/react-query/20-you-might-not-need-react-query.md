# You Might Not Need React Query - Best Practices

## Key Concept

**Every tool should help solve a problem you're having.** React Query is not always necessary, especially with modern frameworks offering first-class data fetching solutions.

## When You Might Not Need React Query

### 1. New Applications with Server Components

**Best Practice**: If you're starting a new app with Server Components (Next.js App Router, Remix), you might not need React Query:

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

**Why**: 
- Server Components fetch data on server
- No client-side async state management needed
- Framework handles data fetching
- React Query is primarily for **client-side** async state

### 2. Framework Has Built-in Data Fetching

**Best Practice**: Use framework's built-in solutions:

- **Next.js**: Server Components, Server Actions
- **Remix**: Loaders, Actions
- **SvelteKit**: Load functions

**Key principle**: If your framework has first-class support for data fetching, use it. Don't add React Query just because.

### 3. Static Data with `staleTime: Infinity`

**Best Practice**: Move static data fetching to Server Components:

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

**Why**: Server Components are better for truly static data that never changes.

## When You Still Need React Query

### 1. Hybrid Approach with Server Components

**Best Practice**: Combine Server Components with React Query for client-side features:

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

**Use cases**:
- Infinite scrolling (prefetch on server, load more on client)
- Offline support
- Background refetching (interval fetching, auto-refetch on focus)
- Optimistic updates
- Real-time updates

### 2. Client-Only Applications

**Best Practice**: React Query is still great for:
- SPAs without server
- React Native apps
- Non-React frameworks (Vue, Solid via TanStack Query)
- Backend not in Node.js

### 3. Advanced Client-Side Features

**Best Practice**: Use React Query for features Server Components don't handle well:

- **Interval fetching**: Polling for updates
- **Smart refetches**: On window focus, reconnect
- **Optimistic updates**: Update UI before server confirms
- **Offline support**: Cache and sync when online
- **Complex state management**: Derived state, transformations

## Best Practices

### 1. Evaluate Your Use Case

**Best Practice**: Ask yourself:
- Is data fetching happening on server? → Maybe not needed
- Do you need client-side caching/refetching? → Probably needed
- Does framework handle your use case? → Use framework
- Need offline support? → React Query helps

### 2. Incremental Adoption

**Best Practice**: You don't have to choose all-or-nothing:

```typescript
// ✅ Hybrid approach
// Some components use Server Components
export default async function StaticPage() {
  const data = await fetchStaticData()
  return <StaticContent data={data} />
}

// Some components use React Query
'use client'
function InteractiveList() {
  const { data } = useQuery({
    queryKey: ['items'],
    queryFn: fetchItems,
  })
  return <List data={data} />
}
```

**Benefits**:
- Move static data to Server Components
- Keep interactive features with React Query
- Incremental migration path

### 3. Use Right Tool for Right Job

**Best Practice**: 
- **Server Components**: Static data, initial page load
- **React Query**: Client-side caching, refetching, offline
- **Framework loaders**: Route-level data fetching (Remix)

**Key principle**: There's no free lunch - everything is a tradeoff. Choose based on your needs.

## Not To Do

### 1. Don't Use React Query Just Because

```typescript
// 🚨 Unnecessary - Server Component handles this
'use client'
function Page() {
  const { data } = useQuery({
    queryKey: ['static-data'],
    queryFn: async () => {
      // This could be a Server Component
      return fetch('/api/static-data')
    },
  })
  return <div>{data}</div>
}

// ✅ Better - use Server Component
export default async function Page() {
  const data = await fetch('/api/static-data')
  return <div>{data}</div>
}
```

### 2. Don't Ignore Framework Solutions

```typescript
// 🚨 Using React Query when Remix loaders exist
export default function Page() {
  const { data } = useQuery({
    queryKey: ['items'],
    queryFn: () => fetch('/api/items'),
  })
  // ...
}

// ✅ Better - use Remix loader
export async function loader() {
  return json({ items: await fetchItems() })
}
```

### 3. Don't Rush to Server Components

**Key insight**: Server Components are still bleeding edge:
- Require framework integration
- Need server infrastructure
- Early in development

**Best Practice**: Don't feel obliged to move everything immediately. Evaluate tradeoffs.

### 4. Don't Use React Query for Everything

```typescript
// 🚨 Overkill for simple static data
useQuery({
  queryKey: ['config'],
  queryFn: fetchConfig,
  staleTime: Infinity,  // Never changes
})

// ✅ Consider Server Component or just fetch once
```

## Key Insights

1. **Every tool solves a problem**: Use React Query when it helps, not just because it exists
2. **Server Components change the game**: For server-side data fetching, they might be enough
3. **Hybrid approach works**: Combine Server Components (static) with React Query (interactive)
4. **Framework solutions first**: Use built-in data fetching when available
5. **No free lunch**: Everything is a tradeoff - evaluate your needs
6. **Incremental adoption**: You don't have to choose all-or-nothing
7. **React Query isn't dead**: Still valuable for client-side features, offline support, and non-React frameworks
