---
description: "React Query with React Router: use loaders to fetch early, React Query to cache, use getQueryData ?? fetchQuery or ensureQueryData"
alwaysApply: false
---

# React Query meets React Router Rules

## Combine Router Loaders with React Query Cache

- **Use router loaders to fetch early, React Query to cache**

```typescript
// ✅ Define query options
const contactDetailQuery = (id) => ({
  queryKey: ['contacts', 'detail', id],
  queryFn: async () => getContact(id),
})

// ✅ Loader prefills cache
export const loader =
  (queryClient) =>
  async ({ params }) => {
    const query = contactDetailQuery(params.contactId)
    // Return cached data or fetch if not available
    return (
      queryClient.getQueryData(query.queryKey) ??
      (await queryClient.fetchQuery(query))
    )
  }

// ✅ Component uses React Query
export default function Contact() {
  const params = useParams()
  const { data: contact } = useQuery(contactDetailQuery(params.contactId))
  // render jsx
}
```

- **Why**:
  - Router fetches **early** (before component renders)
  - React Query **caches** the data
  - Subsequent visits show cached data instantly
  - Background refetch if stale

## Use getQueryData ?? fetchQuery Pattern

- **Check cache first, fetch only if needed**

```typescript
export const loader =
  (queryClient) =>
  async ({ params }) => {
    const query = contactDetailQuery(params.contactId)
    return (
      queryClient.getQueryData(query.queryKey) ??  // ✅ Return cached if available
      (await queryClient.fetchQuery(query))         // ✅ Fetch if not in cache
    )
  }
```

- **Benefits**:
  - Shows cached data immediately on return visits
  - Only fetches when cache is empty
  - Errors thrown to `errorElement` (unlike `prefetchQuery`)

## Use ensureQueryData (v4.18.0+)

- **Built-in helper (same as getQueryData ?? fetchQuery)**

```typescript
// ✅ Built-in helper
export const loader =
  (queryClient) =>
  async ({ params }) => {
    return queryClient.ensureQueryData(
      contactDetailQuery(params.contactId)
    )
  }
```

## Pass QueryClient to Loaders

- **Pass `queryClient` explicitly to loaders**

```typescript
const queryClient = new QueryClient()

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    children: [
      {
        path: 'contacts/:contactId',
        element: <Contact />,
        loader: contactLoader(queryClient),  // ✅ Pass queryClient
      },
    ],
  },
])
```

## Use initialData from Loaders for Type Safety

- **Pass loader data as `initialData` for type safety**

```typescript
export default function Contact() {
  const params = useParams()
  const loaderData = useLoaderData()
  
  const { data: contact } = useQuery({
    ...contactDetailQuery(params.contactId),
    initialData: loaderData,  // ✅ Type-safe from loader
  })
}
```

## Invalidate Queries in Actions

- **Invalidate queries after mutations in router actions**

```typescript
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formData = await request.formData()
    await updateContact(params.contactId, formData)
    
    // ✅ Invalidate queries after mutation
    await queryClient.invalidateQueries({
      queryKey: ['contacts', 'detail', params.contactId],
    })
    
    return redirect(`/contacts/${params.contactId}`)
  }
```

## Control Transition Timing with await

- **Await invalidations to keep mutation in loading state during transition**

```typescript
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    await updateContact(params.contactId, formData)
    
    // ✅ Await invalidation - keeps mutation in loading state
    await queryClient.invalidateQueries({
      queryKey: ['contacts', 'detail', params.contactId],
    })
    
    return redirect(`/contacts/${params.contactId}`)
  }
```

## Don't Do

- **Don't use `prefetchQuery` in loaders**
  - Doesn't throw errors to `errorElement`
  - Fix: Use `getQueryData ?? fetchQuery` or `ensureQueryData`

- **Don't forget to pass `queryClient` to loaders**
  - Loaders need access to query client
  - Fix: Pass `queryClient` explicitly when creating router

- **Don't skip awaiting invalidations in actions**
  - Mutation completes before queries update
  - Fix: Await `invalidateQueries` to keep mutation in loading state
