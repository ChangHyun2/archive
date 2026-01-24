---
description: "React Query testing: use MSW, create new QueryClient per test, turn off retries, use setQueryDefaults, always await queries"
alwaysApply: false
---

# React Query Testing Rules

## Use Mock Service Worker (MSW)

- **Use MSW for mocking network requests**
  - Works in Node for testing
  - Supports REST and GraphQL
  - Has Storybook addon
  - Works in browser for development
  - Works with Cypress
  - Single source of truth for API mocking

## Create New QueryClient for Each Test

- **Create a new `QueryClient` for each test to ensure isolation**

```typescript
// ✅ Creates new QueryClient for each test
const createWrapper = () => {
  const queryClient = new QueryClient()
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

test('my first test', async () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper(),
  })
})
```

## Turn Off Retries in Tests

- **Disable retries to prevent test timeouts**

```typescript
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,  // ✅ Turns retries off
      },
    },
  })
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}
```

## Use setQueryDefaults Instead of Inline Options

- **Don't set options directly on `useQuery` - use defaults**
  - Allows overriding in tests
  - Centralized configuration
  - Can't override if set directly on `useQuery`

```typescript
// ✅ Use setQueryDefaults
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
    },
  },
})

queryClient.setQueryDefaults(['todos'], { retry: 5 })

// ✅ Can still turn off for tests
const testQueryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: false,  // ✅ Overrides all defaults
    },
  },
})
```

## Always Await the Query

- **Wait for query to complete before assertions**

```typescript
// ✅ Wait until query succeeds
test('my first test', async () => {
  const { result, waitFor } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })

  await waitFor(() => result.current.isSuccess)
  expect(result.current.data).toBeDefined()
})
```

## Don't Do

- **Don't set options directly on `useQuery`**
  - Cannot override in tests
  - Fix: Use `setQueryDefaults` or global defaults

- **Don't share QueryClient between tests**
  - Shared state between tests causes flaky tests
  - Fix: Create new `QueryClient` in wrapper for each test

- **Don't forget to turn off retries**
  - Tests will timeout
  - Fix: Set `retry: false` in default options

- **Don't assert immediately**
  - Query is still loading
  - Fix: Use `waitFor` to wait for query completion

- **Don't mock fetch/axios directly**
  - MSW is better - works everywhere, single source of truth
  - Fix: Use MSW for API mocking
