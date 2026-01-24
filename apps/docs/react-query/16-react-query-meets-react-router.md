# React Query meets React Router - Best Practices

## Key Concept

**React Router is not a cache** - it's about **when** to fetch. React Query is about **what** to cache. They work perfectly together.

## Best Practices

### 1. Combine Router Loaders with React Query Cache

**Best Practice**: Use router loaders to fetch early, React Query to cache:

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

**Why**:
- Router fetches **early** (before component renders)
- React Query **caches** the data
- Subsequent visits show cached data instantly
- Background refetch if stale

### 2. Use `getQueryData ?? fetchQuery` Pattern

**Best Practice**: Check cache first, fetch only if needed:

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

**Benefits**:
- Shows cached data immediately on return visits
- Only fetches when cache is empty
- Errors thrown to `errorElement` (unlike `prefetchQuery`)

**Alternative**: Use `ensureQueryData` (v4.18.0+):

```typescript
// ✅ Built-in helper (same as getQueryData ?? fetchQuery)
export const loader =
  (queryClient) =>
  async ({ params }) => {
    return queryClient.ensureQueryData(
      contactDetailQuery(params.contactId)
    )
  }
```

### 3. Pass QueryClient to Loaders

**Best Practice**: Pass `queryClient` explicitly to loaders:

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

**Why**: Loaders are not hooks, so can't use `useQueryClient()`. Passing explicitly is the recommended approach.

### 4. Use `initialData` from Loader for Type Safety

**Best Practice**: Use `useLoaderData` as `initialData` to narrow types:

```typescript
export default function Contact() {
  const initialData = useLoaderData() as Awaited<
    ReturnType<ReturnType<typeof loader>>
  >
  const params = useParams()
  const { data: contact } = useQuery({
    ...contactDetailQuery(params.contactId),
    initialData,  // ✅ TypeScript knows data is not undefined
  })
  // contact is Contact, not Contact | undefined
}
```

**Why**: TypeScript can't know loader data exists, but `initialData` narrows the type.

### 5. Invalidate Queries in Actions

**Best Practice**: Invalidate React Query cache in router actions:

```typescript
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formData = await request.formData()
    const updates = Object.fromEntries(formData)
    await updateContact(params.contactId, updates)
    await queryClient.invalidateQueries({ queryKey: ['contacts'] })  // ✅ Invalidate
    return redirect(`/contacts/${params.contactId}`)
  }
```

**Why**: Router actions invalidate loaders, but we need to invalidate React Query cache too.

### 6. Control Transition Timing with `await`

**Best Practice**: Use `await` as a lever to control when transition happens:

```typescript
// ✅ Fast transition - show stale data, update in background
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formData = await request.formData()
    const updates = Object.fromEntries(formData)
    await updateContact(params.contactId, updates)
    queryClient.invalidateQueries({ queryKey: ['contacts'] })  // ✅ No await
    return redirect(`/contacts/${params.contactId}`)
  }

// ✅ Wait for fresh data - avoid layout shifts
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formData = await request.formData()
    const updates = Object.fromEntries(formData)
    await updateContact(params.contactId, updates)
    await queryClient.invalidateQueries({ queryKey: ['contacts'] })  // ✅ Await
    return redirect(`/contacts/${params.contactId}`)
  }
```

**When to await**:
- **Don't await**: Fast transition, show stale data, update in background
- **Await**: Avoid layout shifts, keep action pending until fresh data

**Mix and match**: Await important invalidations, let others run in background.

## Not To Do

### 1. Don't Use Router Loaders Without Caching

```typescript
// 🚨 Fetches on every visit, even if data is cached
export async function loader({ params }) {
  return getContact(params.contactId)  // ❌ No cache
}

export default function Contact() {
  const contact = useLoaderData()  // Always waits for fetch
}
```

**Problem**: Going back to Contact 1 after visiting Contact 2 will fetch again, even though data was already fetched.

**Solution**: Use React Query cache with loader.

### 2. Don't Use `prefetchQuery` in Loaders

```typescript
// 🚨 prefetchQuery doesn't return data and catches errors
export const loader =
  (queryClient) =>
  async ({ params }) => {
    queryClient.prefetchQuery(contactDetailQuery(params.contactId))  // ❌ No return value
    // Component still needs to wait for useQuery
  }
```

**Problem**: 
- Doesn't return data (loader can't provide it)
- Catches errors internally (won't reach `errorElement`)

**Solution**: Use `fetchQuery` or `ensureQueryData`.

### 3. Don't Import QueryClient Directly

```typescript
// 🚨 Loses dependency injection benefits
export const queryClient = new QueryClient()

export const loader = async ({ params }) => {
  return queryClient.getQueryData(...)  // ❌ Tight coupling
}
```

**Problem**: Can't use different clients for testing, loses flexibility.

**Solution**: Pass `queryClient` as parameter.

### 4. Don't Forget to Invalidate in Actions

```typescript
// 🚨 Cache won't update after mutation
export const action = async ({ request, params }) => {
  const formData = await request.formData()
  const updates = Object.fromEntries(formData)
  await updateContact(params.contactId, updates)
  return redirect(`/contacts/${params.contactId}`)  // ❌ No invalidation
}
```

**Problem**: Router invalidates loaders, but React Query cache stays stale.

**Solution**: Invalidate React Query cache in action.

## Key Insights

1. **Router = when, React Query = what**: Router fetches early, React Query caches
2. **Best of both worlds**: Early fetching + smart caching
3. **getQueryData ?? fetchQuery**: Check cache first, fetch if needed
4. **ensureQueryData**: Built-in helper (v4.18.0+) for same pattern
5. **Pass queryClient**: Explicitly pass to loaders (not hooks)
6. **initialData for types**: Use loader data as `initialData` to narrow types
7. **Invalidate in actions**: Keep React Query cache in sync
8. **await as lever**: Control transition timing based on needs
