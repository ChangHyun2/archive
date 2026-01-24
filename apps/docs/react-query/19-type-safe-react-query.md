# Type-safe React Query - Best Practices

## Key Concept

**Trust is essential for type safety.** To truly leverage TypeScript, we need to trust our type definitions. Manual generics are often "lies" - type assertions in disguise.

## Best Practices

### 1. Type the queryFn, Not useQuery

**Best Practice**: Type the return value of `queryFn` instead of providing generics to `useQuery`:

```typescript
// ✅ Type the function return value
type Todo = { id: number; name: string; done: boolean }

const fetchTodo = async (id: number): Promise<Todo> => {
  const response = await axios.get(`/todos/${id}`)
  return response.data
}

// ✅ No generics on useQuery
const query = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetchTodo(id),
})

// ✅ Types properly inferred
query.data  // Todo | undefined
```

**Why**: 
- React Query infers types from `queryFn` return type
- No need for manual generics
- Type inference flows naturally
- Less TypeScript complexity

### 2. Use Zod for Runtime Validation

**Best Practice**: Validate network responses with Zod schemas:

```typescript
import { z } from 'zod'

// ✅ Define schema (replaces type definition)
const todoSchema = z.object({
  id: z.number(),
  name: z.string(),
  done: z.boolean(),
})

const fetchTodo = async (id: number) => {
  const response = await axios.get(`/todos/${id}`)
  // ✅ Parse against schema (replaces type assertion)
  return todoSchema.parse(response.data)
}

const query = useQuery({
  queryKey: ['todos', id],
  queryFn: () => fetchTodo(id),
})
```

**Benefits**:
- Runtime validation (not just compile-time)
- Throws descriptive error if data doesn't match
- React Query goes to error state (same as network failure)
- Types inferred from schema automatically
- No surprises for users - errors handled properly

**Key principle**: "The more your TypeScript code looks like JavaScript, the better."

### 3. Design Resilient Schemas

**Best Practice**: Make schemas resilient to avoid false failures:

```typescript
// ✅ Resilient schema
const todoSchema = z.object({
  id: z.number(),
  name: z.string(),
  done: z.boolean(),
  // ✅ Optional if it doesn't matter
  tags: z.array(z.string()).optional(),
  // ✅ Handle null/undefined gracefully
  description: z.string().nullable(),
})
```

**Why**: 
- If optional property is null/undefined, don't fail the query
- Better UX - only fail on truly invalid data
- Matches real-world API behavior

### 4. Use queryOptions for Type-Safe getQueryData (v5+)

**Best Practice**: Use `queryOptions` to make `getQueryData` type-safe:

```typescript
// ✅ v5+ - type-safe cache access
const todosQuery = queryOptions({
  queryKey: ['todos', id],
  queryFn: () => fetchTodo(id),
})

const todo = queryClient.getQueryData(todosQuery.queryKey)
//    ^? Todo | undefined (type inferred!)
```

**Why**: `queryOptions` tags the `queryKey` with type information, enabling type inference.

## Not To Do

### 1. Don't Provide Manual Generics to useQuery

```typescript
// 🚨 Manual generic - only one of four generics
const query = useQuery<Todo>({
  queryKey: ['todos', id],
  queryFn: () => fetchTodo(id),
})
```

**Problems**:
- `useQuery` has 4 generics - providing one leaves others as defaults
- Unnecessary complexity
- Often a "lie" (type assertion in disguise)

**Fix**: Type the `queryFn` return value instead.

### 2. Don't Use "Return-Only" Generics

```typescript
// 🚨 Violates golden rule of Generics
const fetchTodo = async (id: number) => {
  const response = await axios.get<Todo>(`/todos/${id}`)
  // ❌ Generic only appears in return type
  return response.data
}
```

**Golden Rule of Generics**: "For a Generic to be useful, it must appear at least twice."

**Why it's a lie**: 
- Generic `T` only appears in return type
- Same as type assertion: `response.data as Todo`
- No actual type safety

**Fix**: Use explicit type assertion (at least it's honest) or validate with Zod.

### 3. Don't Trust Type Assertions

```typescript
// 🚨 Type assertion - bypasses compiler
const fetchTodo = async (id: number) => {
  const response = await axios.get(`/todos/${id}`)
  return response.data as Todo  // ❌ Unsafe
}
```

**Problem**: 
- Bypasses TypeScript compiler
- No runtime validation
- If backend returns wrong shape, error appears later in different place
- Users see "cannot read property X of undefined"

**Fix**: Use Zod validation for runtime safety.

### 4. Don't Forget to Handle getQueryData Type Safety

```typescript
// 🚨 Returns unknown
const todo = queryClient.getQueryData(['todos', 1])
//    ^? unknown

// 🚨 Manual generic (type assertion)
const todo = queryClient.getQueryData<Todo>(['todos', 1])
//    ^? Todo | undefined
```

**Problem**: React Query can't know what's in cache without upfront definition.

**Fix**: 
- Use `queryOptions` (v5+) for type-safe access
- Or validate with Zod if needed
- Or use tools like `react-query-kit`

## Key Insights

1. **Trust is essential**: We need to trust type definitions for them to be useful
2. **Type queryFn, not useQuery**: Let inference work, avoid manual generics
3. **Zod for validation**: Runtime validation + type inference = true type safety
4. **Golden rule of Generics**: Generic must appear at least twice to be useful
5. **Return-only generics are lies**: Same as type assertions, just hidden
6. **Resilient schemas**: Design schemas to match real API behavior
7. **queryOptions (v5+)**: Makes `getQueryData` type-safe
8. **End-to-end type safety**: Consider tRPC or zodios for full-stack type safety
