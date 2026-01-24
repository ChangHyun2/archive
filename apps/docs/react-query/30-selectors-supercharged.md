# React Query Selectors, Supercharged - Best Practices

## Important Disclaimer

**`select` is an optimization** you probably don't need when starting out with React Query on a small application. It's an advanced feature for fine-grained subscriptions.

## Key Concepts

### Fine-grained Subscriptions

React Query uses `QueryHash` to filter subscriptions - components only get notified about changes to their specific Query. However, sometimes endpoints return a lot of data, and you only need specific fields.

**`select`** allows you to pick, transform, or compute a result that your component should subscribe to. It's similar to selectors in Redux or Zustand.

## Best Practices

### 1. Use `select` for Fine-grained Subscriptions

**Best Practice**: Use `select` to subscribe only to fields you need:

```typescript
function ProductTitle({ id }: Props) {
  const productTitleQuery = useSuspenseQuery({
    ...productOptions(id),
    select: (data) => data.title,  // ✅ Only subscribe to title
  })

  return <h1>{productTitleQuery.data}</h1>
}
```

**Why**: Component only re-renders if `title` changes, even if other product properties (like purchase count or comments) change frequently.

### 2. Pick Multiple Properties with Structural Sharing

**Best Practice**: You can pick multiple properties - React Query uses structural sharing on the select result:

```typescript
function Product({ id }: Props) {
  const productQuery = useSuspenseQuery({
    ...productOptions(id),
    select: (data) => ({
      title: data.title,
      description: data.description,
    }),
  })

  return (
    <main>
      <h1>{productQuery.data.title}</h1>
      <p>{productQuery.data.description}</p>
    </main>
  )
}
```

**Why**: If either `title` or `description` changes, you get a re-render. Otherwise not. React Query handles referential stability automatically.

### 3. Stabilize `select` with `useCallback` for Expensive Transformations

**Best Practice**: Use `useCallback` to stabilize the `select` function when it has dependencies:

```typescript
function ProductList({ filters, minRating }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select: React.useCallback(
      (data) => expensiveSuperTransformation(data, minRating),
      [minRating]  // ✅ Stable unless minRating changes
    ),
  })

  return (
    <ul>
      {productsQuery.data.map((product) => (
        <li>{product.summary}</li>
      ))}
    </ul>
  )
}
```

**Why**: React Query tracks referential identity of `select`. If it gets "the same function", it can skip re-running it. Inline functions are always new, so this optimization doesn't apply.

### 4. Move Stable `select` Functions Outside Component

**Best Practice**: If `select` has no dependencies, move it outside the component:

```typescript
// ✅ Stable reference - no useCallback needed
const select = (data: Array<Product>) =>
  expensiveSuperTransformation(data)

function ProductList({ filters }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select,
  })

  return (
    <ul>
      {productsQuery.data.map((product) => (
        <li>{product.summary}</li>
      ))}
    </ul>
  )
}
```

**Why**: No need for `useCallback` with empty dependency array - function is already stable.

### 5. Use External Memoization for Multiple Observers

**Best Practice**: For expensive transformations used by multiple components, use external memoization (e.g., `fast-memoize`):

```typescript
import memoize from 'fast-memoize'

// ✅ Memoized at module level
const select = memoize((data: Array<Product>) =>
  expensiveSuperTransformation(data)
)

function ProductList({ filters }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select,
  })

  return (
    <ul>
      {productsQuery.data.map((product) => (
        <li>{product.summary}</li>
      ))}
    </ul>
  )
}
```

**Why**: 
- Each `useQuery` call creates a `QueryObserver` that caches `select` result per observer
- If you render the same component 3 times, `select` runs 3 times (once per observer)
- External memoization ensures `expensiveSuperTransformation` only runs once per unique data
- This is as optimized as you can get

### 6. Type Select Abstractions (If You Must)

**Best Practice**: If you need to abstract `select` in a reusable function, use generics:

```typescript
const productOptions = <TData = ProductData>(
  id: string,
  select?: (data: ProductData) => TData
) => {
  return queryOptions({
    queryKey: ['product', id],
    queryFn: () => fetchProduct(id),
    select,
  })
}
```

**Note**: The recommended approach is to use the Query Options API and specify `select` at the usage site, not in abstractions.

## Not To Do

### 1. Don't Use `select` Prematurely

```typescript
// 🚨 Probably unnecessary for simple cases
function ProductTitle({ id }: Props) {
  const productQuery = useSuspenseQuery({
    ...productOptions(id),
    select: (data) => data.title,  // ❌ Might be overkill
  })
  return <h1>{productQuery.data}</h1>
}
```

**Why**: For small apps or simple components, occasional re-renders aren't a problem. Only optimize when you have a proven performance issue.

### 2. Don't Use Inline Functions for Expensive Transformations

```typescript
// 🚨 Runs on every render
function ProductList({ filters }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select: (data) => expensiveSuperTransformation(data),  // ❌ New function every render
  })
}
```

**Why**: Inline functions are always newly created, so React Query can't optimize by skipping re-runs.

**Solution**: Use `useCallback` or move function outside component.

### 3. Don't Rely on Shared Mutable State in `select`

```typescript
// 🚨 Don't do this
let externalState = 0

const select = (data) => {
  externalState++  // ❌ Shared mutable state
  return data.map(item => ({ ...item, count: externalState }))
}
```

**Why**: React Query assumes same function = same result. Shared mutable state breaks this assumption.

### 4. Don't Manually Provide Generic Types

```typescript
// 🚨 Breaks type inference
const productQuery = useQuery<ProductData, Error>({
  queryKey: ['product', id],
  queryFn: () => fetchProduct(id),
  select: (data) => data.title,  // Type won't be inferred correctly
})
```

**Why**: Type inference works automatically if you don't provide generics. Let TypeScript do its magic.

## Key Insights

1. **`select` enables fine-grained subscriptions**: Subscribe only to fields you need, not the entire query result
2. **Structural sharing works on select results**: You can pick multiple properties without worrying about referential stability
3. **Stabilize `select` functions**: Use `useCallback` or move outside component to enable React Query's optimization
4. **External memoization for multiple observers**: Use libraries like `fast-memoize` when the same transformation is used by multiple components
5. **React Query caches per observer**: Each `useQuery` call creates a `QueryObserver` that caches the `select` result independently
6. **Don't over-optimize**: `select` is an advanced feature - only use when you have proven performance needs
