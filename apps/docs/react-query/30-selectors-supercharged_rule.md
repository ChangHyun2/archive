---
description: "React Query selectors: use select for fine-grained subscriptions, stabilize with useCallback for expensive transformations, move stable functions outside component"
alwaysApply: false
---

# React Query Selectors Rules

## Use `select` for Fine-grained Subscriptions

- **Use `select` to subscribe only to fields you need**

```typescript
function ProductTitle({ id }: Props) {
  const productTitleQuery = useSuspenseQuery({
    ...productOptions(id),
    select: (data) => data.title,  // ✅ Only subscribe to title
  })

  return <h1>{productTitleQuery.data}</h1>
}
```

- **Why**: Component only re-renders if `title` changes, even if other product properties (like purchase count or comments) change frequently.

## Pick Multiple Properties with Structural Sharing

- **You can pick multiple properties - React Query uses structural sharing on the select result**

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

- **Why**: If either `title` or `description` changes, you get a re-render. Otherwise not. React Query handles referential stability automatically.

## Stabilize `select` with `useCallback` for Expensive Transformations

- **Use `useCallback` to stabilize the `select` function when it has dependencies**

```typescript
function ProductList({ filters, minRating }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select: React.useCallback(
      (data) => expensiveSuperTransformation(data, minRating),
      [minRating]  // ✅ Stable unless minRating changes
    ),
  })
}
```

- **Why**: React Query tracks referential identity of `select`. If it gets "the same function", it can skip re-running it. Inline functions are always new, so this optimization doesn't apply.

## Move Stable `select` Functions Outside Component

- **If `select` has no dependencies, move it outside the component**

```typescript
// ✅ Stable reference - no useCallback needed
const select = (data: Array<Product>) =>
  expensiveSuperTransformation(data)

function ProductList({ filters }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select,
  })
}
```

## Use External Memoization for Multiple Observers

- **If multiple components use the same expensive transformation, use external memoization**

```typescript
// ✅ Shared memoization across observers
const memoizedTransform = memoize((data: Array<Product>) =>
  expensiveSuperTransformation(data)
)

function ProductList({ filters }: Props) {
  const productsQuery = useSuspenseQuery({
    ...productListOptions(filters),
    select: memoizedTransform,
  })
}
```

## Type `select` Abstractions with Generics

- **Use generics to type `select` abstractions**

```typescript
function useProductQuery<TSelected = Product>(
  id: number,
  select?: (data: Product) => TSelected
) {
  return useQuery({
    ...productOptions(id),
    select,
  })
}
```

## Don't Do

- **Don't use `select` prematurely**
  - `select` is an optimization you probably don't need when starting out
  - Fix: Only use when you have proven performance issues

- **Don't use inline functions for expensive transformations**
  - Inline functions are always new, so optimization doesn't apply
  - Fix: Use `useCallback` or move function outside component

- **Don't rely on shared mutable state in `select`**
  - `select` should be pure function
  - Fix: Pass all dependencies through function parameters or use `useCallback` dependencies
