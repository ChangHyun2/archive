---
description: "Type-safe React Query: type queryFn not useQuery, use Zod for runtime validation, use queryOptions for type-safe cache access"
alwaysApply: false
---

# Type-safe React Query Rules

## Type the queryFn, Not useQuery

- **Type the return value of `queryFn` instead of providing generics to `useQuery`**

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

## Use Zod for Runtime Validation

- **Validate network responses with Zod schemas**

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

- **Benefits**:
  - Runtime validation (not just compile-time)
  - Throws descriptive error if data doesn't match
  - React Query goes to error state (same as network failure)
  - Types inferred from schema automatically

## Design Resilient Schemas

- **Make schemas resilient to avoid false failures**

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

- **Why**: If optional property is null/undefined, don't fail the query. Better UX - only fail on truly invalid data.

## Use queryOptions for Type-Safe getQueryData (v5+)

- **Use `queryOptions` to make `getQueryData` type-safe**

```typescript
// ✅ v5+ - type-safe cache access
const todosQuery = queryOptions({
  queryKey: ['todos', id],
  queryFn: () => fetchTodo(id),
})

const todo = queryClient.getQueryData(todosQuery.queryKey)
//    ^? Todo | undefined (type inferred!)
```

## Don't Do

- **Don't provide manual generics to `useQuery`**
  - `useQuery` has 4 generics - providing one leaves others as defaults
  - Unnecessary complexity
  - Often a "lie" (type assertion in disguise)
  - Fix: Type the `queryFn` return value instead

- **Don't use "return-only" generics**
  - Violates golden rule of Generics: "For a Generic to be useful, it must appear at least twice"
  - Same as type assertion: `response.data as Todo`
  - Fix: Use explicit type assertion (at least it's honest) or validate with Zod

- **Don't trust type assertions**
  - Bypasses TypeScript compiler
  - No runtime validation
  - Fix: Use Zod validation for runtime safety

- **Don't forget to handle `getQueryData` type safety**
  - Returns `unknown` without type parameter
  - Fix: Use `queryOptions` (v5+) for type-safe access
