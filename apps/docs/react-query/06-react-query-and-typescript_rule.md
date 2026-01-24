---
description: "React Query TypeScript: let TypeScript infer types, type queryFn not useQuery, handle errors with instanceof, use skipToken for enabled"
alwaysApply: false
---

# React Query TypeScript Rules

## Let TypeScript Infer Types

- **Type the `queryFn` return value, let React Query infer the rest**

```typescript
// ✅ Type the function return value
function fetchGroups(): Promise<Group[]> {
  return axios.get('groups').then((response) => response.data)
}

// ✅ No generics needed - types inferred
function useGroups() {
  return useQuery({ 
    queryKey: ['groups'], 
    queryFn: fetchGroups 
  })
  // data: Group[] | undefined (inferred!)
}

// ✅ Works with select too
function useGroupCount() {
  return useQuery({
    queryKey: ['groups'],
    queryFn: fetchGroups,
    select: (groups) => groups.length,
  })
  // data: number | undefined (inferred!)
}
```

## Handle Errors with Type Narrowing

- **Use `instanceof` to narrow error type**

```typescript
const groups = useGroups()

// ✅ Use instanceof to narrow type
if (groups.error instanceof Error) {
  return <div>An error occurred: {groups.error.message}</div>
}
```

- **v5+: Register global error type via module augmentation**

```typescript
declare module '@tanstack/react-query' {
  interface Register {
    defaultError: AxiosError  // ✅ Global error type
  }
}
```

## Avoid Destructuring for Type Narrowing

- **Keep query object intact for type narrowing (TypeScript < 4.6)**
  - TypeScript 4.6+ supports control flow analysis for destructured discriminated unions

```typescript
// ✅ Keep whole object
const groupsQuery = useGroups()
if (groupsQuery.isSuccess) {
  // ✅ groupsQuery.data is Group[] (narrowed)
  return <GroupsList data={groupsQuery.data} />
}
```

## Handle `enabled` Option with Type Safety

- **Accept `undefined` in `queryFn` and reject Promise**

```typescript
// ✅ Handle undefined in queryFn
function fetchGroup(id: number | undefined): Promise<Group> {
  return typeof id === 'undefined'
    ? Promise.reject(new Error('Invalid id'))
    : axios.get(`group/${id}`).then((response) => response.data)
}

function useGroup(id: number | undefined) {
  return useQuery({
    queryKey: ['group', id],
    queryFn: () => fetchGroup(id),
    enabled: Boolean(id),
  })
}
```

- **v5.25+: Use `skipToken` for better type safety**

```typescript
import { useQuery, skipToken } from '@tanstack/query'

function useGroup(id: number | undefined) {
  return useQuery({
    queryKey: ['group', id],
    queryFn: id ? () => fetchGroup(id) : skipToken,  // ✅ Type-safe
  })
}
```

## Type `pageParam` in useInfiniteQuery

- **v5+: Use `initialPageParam` - typed correctly**

```typescript
const queryInfo = useInfiniteQuery({
  queryKey: ['groups'],
  queryFn: ({ pageParam }) => fetchGroups(groups, pageParam),
  getNextPageParam: (lastGroup) => lastGroup.next,
  initialPageParam: 0,  // ✅ Typed as number
})
```

## Narrow Types in defaultQueryFn

- **Use runtime checks to narrow `queryKey` types**

```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: async ({ queryKey: [url] }) => {
        // ✅ Narrow type with typeof check
        if (typeof url === 'string') {
          const { data } = await axios.get(`${baseUrl}/${url.toLowerCase()}`)
          return data
        }
        throw new Error('Invalid QueryKey')
      },
    },
  },
})
```

## Don't Do

- **Don't provide manual generics unnecessarily**
  - Must provide all 4 generics if providing any
  - Doesn't work well with `select` option
  - Fix: Type the `queryFn` return value instead

- **Don't provide partial generics**
  - Breaks with `select` option
  - Fix: Provide all 4 generics, or let TypeScript infer

- **Don't inline `queryFn` without types**
  - Data will be `any`
  - Fix: Extract to typed function or add return type annotation

- **Don't assume error is always Error type**
  - Error might be `unknown` (v3) or anything throwable
  - Fix: Use `instanceof Error` check to narrow type

- **Don't ignore `enabled` type safety**
  - `enabled` doesn't perform type narrowing
  - Fix: Handle `undefined` in `queryFn` or use `skipToken` (v5.25+)
