# Testing React Query - Best Practices

## Key Concept

Testing components that use React Query requires setting up the proper environment (QueryClientProvider) and handling async behavior correctly.

## Best Practices

### 1. Use Mock Service Worker (MSW)

**Best Practice**: Use MSW for mocking network requests:

**Why MSW is recommended**:
- Works in Node for testing
- Supports REST and GraphQL
- Has Storybook addon
- Works in browser for development
- Works with Cypress

**Benefits**: Single source of truth for API mocking.

### 2. Create New QueryClient for Each Test

**Best Practice**: Create a new `QueryClient` for each test to ensure isolation:

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

**Why**: 
- Tests are completely isolated
- No shared state between tests
- Prevents flaky tests when running in parallel
- Better than clearing cache after each test

### 3. Turn Off Retries in Tests

**Best Practice**: Disable retries to prevent test timeouts:

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

**Why**: 
- Default is 3 retries with exponential backoff
- Tests will timeout if testing erroneous queries
- Turning off retries makes tests faster and more predictable

**Important**: This only works if `useQuery` doesn't have explicit `retry` set. Explicit options take precedence over defaults.

### 4. Use `setQueryDefaults` Instead of Inline Options

**Best Practice**: Don't set options directly on `useQuery` - use defaults:

```typescript
// ✅ Use setQueryDefaults
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 2,
    },
  },
})

// ✅ Only todos will retry 5 times
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

**Why**: 
- Allows overriding in tests
- Centralized configuration
- Can't override if set directly on `useQuery`

### 5. Always Await the Query

**Best Practice**: Wait for query to complete before assertions:

```typescript
// ✅ Wait until query succeeds
test('my first test', async () => {
  const { result, waitFor } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })

  // ✅ Wait until the query has transitioned to success state
  await waitFor(() => result.current.isSuccess)

  expect(result.current.data).toBeDefined()
})
```

**With @testing-library/react v13.1.0+**:

```typescript
import { waitFor, renderHook } from '@testing-library/react'

test('my first test', async () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })

  // ✅ Return a Promise via expect
  await waitFor(() => expect(result.current.isSuccess).toBe(true))

  expect(result.current.data).toBeDefined()
})
```

**Why**: React Query is async - queries start in loading state. Must wait for completion.

### 6. Create Wrapper for Component Tests

**Best Practice**: Create a wrapper function for component tests:

```typescript
// ✅ Wrapper for component tests
const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: {
      queries: {
        retry: false,
      },
    },
  })
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

// ✅ Use in component tests
const { getByText } = render(<MyComponent />, {
  wrapper: createWrapper(),
})
```

## Not To Do

### 1. Don't Set Options Directly on useQuery

```typescript
// 🚨 Cannot override in tests
function Example() {
  const queryInfo = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
    retry: 5,  // ❌ Hard-coded, can't override
  })
}
```

**Fix**: Use `setQueryDefaults` or global defaults.

### 2. Don't Share QueryClient Between Tests

```typescript
// 🚨 Shared state between tests
const queryClient = new QueryClient()  // ❌ Created once

test('test 1', () => {
  // Uses shared client
})

test('test 2', () => {
  // Might see data from test 1
})
```

**Fix**: Create new `QueryClient` in wrapper for each test.

### 3. Don't Forget to Turn Off Retries

```typescript
// 🚨 Tests will timeout
const createWrapper = () => {
  const queryClient = new QueryClient()  // ❌ Default retries: 3
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}

test('error case', async () => {
  // ❌ Will retry 3 times, test times out
})
```

**Fix**: Set `retry: false` in default options.

### 4. Don't Assert Immediately

```typescript
// 🚨 Query is still loading
test('my test', () => {
  const { result } = renderHook(() => useCustomHook(), {
    wrapper: createWrapper()
  })

  expect(result.current.data).toBeDefined()  // ❌ Still loading!
})
```

**Fix**: Use `waitFor` to wait for query completion.

### 5. Don't Mock fetch/axios Directly

```typescript
// 🚨 Not recommended
jest.mock('axios', () => ({
  get: jest.fn(),
}))
```

**Why**: MSW is better - works everywhere, single source of truth.

**Fix**: Use MSW for API mocking.

## Key Insights

1. **MSW for mocking**: Single source of truth for API mocks
2. **New QueryClient per test**: Ensures test isolation
3. **Turn off retries**: Prevents test timeouts
4. **Use defaults**: Allows overriding in tests via `setQueryDefaults`
5. **Always await**: Queries are async, must wait for completion
6. **Wrapper pattern**: Reusable setup for both hooks and components
7. **Test isolation**: Critical for parallel test execution
