# React Query and TypeScript - Best Practices

## Key Concept

**Let TypeScript infer types** - don't manually provide generics unless necessary. Type inference works best when you type the `queryFn` return value.

## Understanding useQuery Generics

### The Four Generics

```typescript
useQuery<
  TQueryFnData = unknown,    // Type returned from queryFn
  TError = unknown,           // Type of errors from queryFn
  TData = TQueryFnData,       // Type of data property (different if using select)
  TQueryKey extends QueryKey = QueryKey  // Type of queryKey
>
```

**Key insight**: All generics have defaults. If you provide one, you must provide all (no partial type argument inference in TypeScript yet).

## Best Practices

### 1. Let TypeScript Infer Types

**Best Practice**: Type the `queryFn` return value, let React Query infer the rest:

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

**Benefits**:
- No manual generics
- Works for all 4 generics
- Will continue working if more generics added
- Code looks more like JavaScript

### 2. Handle Errors with Type Narrowing

**Best Practice**: Use `instanceof` to narrow error type:

```typescript
const groups = useGroups()

// ✅ Use instanceof to narrow type
if (groups.error instanceof Error) {
  return <div>An error occurred: {groups.error.message}</div>
}
```

**Why**: 
- Error defaults to `unknown` (v3) or `Error` (v4+)
- In JavaScript, you can throw anything (`throw 5`, `throw undefined`)
- `instanceof` check narrows type and ensures `message` exists at runtime

**Update (v4+)**: Error defaults to `Error` instead of `unknown` for easier usage.

**Update (v5+)**: Can register global error type via module augmentation:

```typescript
declare module '@tanstack/react-query' {
  interface Register {
    defaultError: AxiosError  // ✅ Global error type
  }
}
```

### 3. Avoid Destructuring for Type Narrowing

**Best Practice**: Keep query object intact for type narrowing:

```typescript
// ✅ Keep whole object
const groupsQuery = useGroups()
if (groupsQuery.isSuccess) {
  // ✅ groupsQuery.data is Group[] (narrowed)
  return <GroupsList data={groupsQuery.data} />
}

// 🚨 Destructuring breaks narrowing (TypeScript < 4.6)
const { data, isSuccess } = useGroups()
if (isSuccess) {
  // data is still Group[] | undefined
}
```

**Update**: TypeScript 4.6+ supports control flow analysis for destructured discriminated unions, so this works now.

**Why avoid destructuring anyway**:
- Names like `data` and `error` are universal - you'll rename them
- Keeping whole object keeps context
- Helps TypeScript narrow types

### 4. Handle `enabled` Option with Type Safety

**Best Practice**: Accept `undefined` in `queryFn` and reject Promise:

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

**Why**: `enabled` doesn't perform type narrowing. `refetch` can bypass it, so `id` might be `undefined`.

**Update (v5.25+)**: Use `skipToken` for better type safety:

```typescript
import { useQuery, skipToken } from '@tanstack/query'

function useGroup(id: number | undefined) {
  return useQuery({
    queryKey: ['group', id],
    queryFn: id ? () => fetchGroup(id) : skipToken,  // ✅ Type-safe
  })
}
```

### 5. Type `pageParam` in useInfiniteQuery

**Best Practice**: Explicitly type `pageParam` to override `any`:

```typescript
type GroupResponse = { next?: number; groups: Group[] }

const queryInfo = useInfiniteQuery({
  queryKey: ['groups'],
  queryFn: ({
    pageParam = 0,
  }: {
    pageParam: GroupResponse['next']  // ✅ Explicitly type
  }) => fetchGroups(groups, pageParam),
  getNextPageParam: (lastGroup) => lastGroup.next,
})
```

**Update (v5+)**: Use `initialPageParam` - typed correctly:

```typescript
const queryInfo = useInfiniteQuery({
  queryKey: ['groups'],
  queryFn: ({ pageParam }) => fetchGroups(groups, pageParam),
  getNextPageParam: (lastGroup) => lastGroup.next,
  initialPageParam: 0,  // ✅ Typed as number
})
```

### 6. Narrow Types in defaultQueryFn

**Best Practice**: Use runtime checks to narrow `queryKey` types:

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

**Why**: `queryKey` is `unknown[]` at `QueryClient` creation time - no guarantee how keys will be constructed. Must narrow with runtime checks.

## Not To Do

### 1. Don't Provide Manual Generics Unnecessarily

```typescript
// 🚨 Unnecessary - let inference work
function useGroups() {
  return useQuery<Group[], Error>({
    queryKey: ['groups'],
    queryFn: fetchGroups,
  })
}

// ✅ Better - type the function
function fetchGroups(): Promise<Group[]> { ... }
function useGroups() {
  return useQuery({ queryKey: ['groups'], queryFn: fetchGroups })
}
```

**Problem**: 
- Must provide all 4 generics if providing any
- Doesn't work well with `select` option
- More complex code

### 2. Don't Provide Partial Generics

```typescript
// 🚨 Breaks with select option
function useGroupCount() {
  return useQuery<Group[], Error>({  // ❌ Only 2 of 4 generics
    queryKey: ['groups'],
    queryFn: fetchGroups,
    select: (groups) => groups.length,  // ❌ Type error!
  })
}
```

**Problem**: Third generic defaults to `Group[]`, but `select` returns `number`.

**Fix**: Provide all 4 generics, or let TypeScript infer.

### 3. Don't Inline queryFn Without Types

```typescript
// 🚨 data will be `any`
function useGroups() {
  return useQuery({
    queryKey: ['groups'],
    queryFn: () =>
      axios.get('groups').then((response) => response.data),  // ❌ any
  })
}
```

**Fix**: Extract to typed function or add return type annotation.

### 4. Don't Assume Error is Always Error Type

```typescript
// 🚨 Error might be unknown (v3) or anything throwable
const groups = useGroups()

if (groups.error) {
  return <div>{groups.error.message}</div>  // ❌ Type error
}
```

**Fix**: Use `instanceof Error` check to narrow type.

### 5. Don't Use Destructuring with Status Checks (TypeScript < 4.6)

```typescript
// 🚨 Type narrowing doesn't work (TS < 4.6)
const { data, isSuccess } = useGroups()
if (isSuccess) {
  // data is still Group[] | undefined
}
```

**Fix**: Keep whole object, or upgrade to TypeScript 4.6+.

### 6. Don't Ignore `enabled` Type Safety

```typescript
// 🚨 Type error - id might be undefined
function useGroup(id: number | undefined) {
  return useQuery({
    queryKey: ['group', id],
    queryFn: () => fetchGroup(id),  // ❌ id might be undefined
    enabled: Boolean(id),
  })
}
```

**Fix**: Handle `undefined` in `queryFn` or use `skipToken` (v5.25+).

## Key Insights

1. **Let TypeScript infer**: Type the `queryFn`, not `useQuery`
2. **Four generics**: `TQueryFnData`, `TError`, `TData`, `TQueryKey`
3. **No partial inference**: Must provide all generics if providing any
4. **Error is unknown/Error**: Use `instanceof` to narrow
5. **Avoid destructuring**: Keep whole object for type narrowing (TS < 4.6)
6. **enabled doesn't narrow**: Handle `undefined` in `queryFn` or use `skipToken`
7. **pageParam is any**: Explicitly type it (or use `initialPageParam` in v5+)
8. **defaultQueryFn**: Narrow `queryKey` types with runtime checks
9. **unknown is great**: Better than `any` - forces defensive programming
