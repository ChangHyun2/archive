# React Query and React Context - Best Practices

## Key Concept

**React Context is a dependency injection tool**, not a state manager. Use it to make implicit dependencies explicit when you know data exists but TypeScript can't verify it.

## The Problem: Implicit Dependencies

### When Direct Query Access Works

**Best Practice**: For self-contained components, use queries directly:

```typescript
// ✅ Self-contained component - handles loading/error
function ProductTable() {
  const productQuery = useProductQuery()

  if (productQuery.data) {
    return <table>...</table>
  }

  if (productQuery.isError) {
    return <ErrorMessage error={productQuery.error} />
  }

  return <SkeletonLoader />
}
```

**Why**: Component is resilient, handles all states, can be moved anywhere.

### When Direct Query Access Fails

**Problem**: Components that know data exists but TypeScript doesn't:

```typescript
// 🚨 TypeScript error - data might be undefined
function UserNameDisplay() {
  const { data } = useCurrentUserQuery()  // data: User | undefined
  return <div>User: {data.userName}</div>  // ❌ Error!
}
```

**Why this is problematic**:
- We know data exists (used earlier in tree)
- TypeScript can't verify this
- Creates implicit dependency (only in our head)
- Error-prone to refactoring
- Might break if component moved

## Best Practices

### 1. Use React Context for Explicit Dependencies

**Best Practice**: Make implicit dependencies explicit with Context:

```typescript
// ✅ Create context
const CurrentUserContext = React.createContext<User | null>(null)

// ✅ Provider handles loading/error, provides data
export const CurrentUserContextProvider = ({
  children,
}: {
  children: React.ReactNode
}) => {
  const currentUserQuery = useCurrentUserQuery()

  if (currentUserQuery.isPending) {
    return <SkeletonLoader />
  }

  if (currentUserQuery.isError) {
    return <ErrorMessage error={currentUserQuery.error} />
  }

  return (
    <CurrentUserContext.Provider value={currentUserQuery.data}>
      {children}
    </CurrentUserContext.Provider>
  )
}

// ✅ Consumer knows data exists
function UserNameDisplay() {
  const data = useCurrentUserContext()  // ✅ Type-safe
  return <div>User: {data.username}</div>
}
```

**Benefits**:
- Makes dependency explicit (visible in code)
- Type-safe access
- Clear contract: Provider must exist
- Refactoring-safe (moving Provider affects all consumers)

### 2. Add Invariant for Type Safety

**Best Practice**: Throw error if context accessed outside Provider:

```typescript
export const useCurrentUserContext = () => {
  const currentUser = React.useContext(CurrentUserContext)
  if (!currentUser) {
    throw new Error('CurrentUserContext: No value provided')  // ✅ Fail fast
  }

  return currentUser  // ✅ TypeScript knows it's User, not User | null
}
```

**Why**:
- TypeScript infers `User` (not `User | null`)
- Fails fast with clear error message
- Prevents silent bugs

### 3. Use `useSuspenseQuery` (v5+)

**Best Practice**: For guaranteed data, use `useSuspenseQuery`:

```typescript
// ✅ v5+ - data is guaranteed to be defined
function UserNameDisplay() {
  const { data } = useSuspenseQuery({
    queryKey: ['user', id],
    queryFn: () => fetchUser(id),
  })
  return <div>User: {data.username}</div>  // ✅ TypeScript happy
}
```

**Why**: 
- `useSuspenseQuery` guarantees data exists when component renders
- TypeScript narrows type automatically
- No need for Context in this case

**Note**: In v4, `suspense: true` doesn't narrow types because query can be disabled.

## Not To Do

### 1. Don't Use Context for State Management

```typescript
// 🚨 Context + useState = poor state management
const StateContext = React.createContext()

function Provider({ children }) {
  const [state, setState] = useState({})
  return (
    <StateContext.Provider value={{ state, setState }}>
      {children}
    </StateContext.Provider>
  )
}
```

**Why**: 
- Performance issues (all consumers re-render)
- Use dedicated state management (Zustand, Redux) instead
- Context is for dependency injection, not state

### 2. Don't Use Type Assertions (`!`)

```typescript
// 🚨 Silences TypeScript but unsafe
function UserNameDisplay() {
  const { data } = useCurrentUserQuery()
  return <div>User: {data!.userName}</div>  // ❌ Unsafe
}
```

**Why**: 
- Ignores potential undefined
- Can cause runtime errors
- Hides real problems

### 3. Don't Add Guards Everywhere

```typescript
// 🚨 Repetitive, error-prone
function UserNameDisplay() {
  const { data } = useCurrentUserQuery()
  if (!data) return null  // ❌ Unnecessary check
  return <div>User: {data.userName}</div>
}
```

**Why**: 
- Litters codebase with "never happens" checks
- Still error-prone if component moved
- Doesn't solve the root problem

### 4. Don't Create Implicit Dependencies

```typescript
// 🚨 Implicit dependency - only in our head
function UserNameDisplay() {
  // We "know" this was called earlier, but code doesn't show it
  const { data } = useCurrentUserQuery()
  return <div>User: {data?.userName}</div>  // ❌ Unsafe
}
```

**Why**: 
- Not visible in code
- Breaks on refactoring
- Colleagues might not know
- Future changes might break it

## Key Insights

1. **Context = Dependency Injection**: Not state management, but dependency injection
2. **Make dependencies explicit**: Use Context when you know data exists but TypeScript doesn't
3. **Provider handles states**: Provider checks loading/error, provides data to children
4. **Invariant for types**: Throw error if accessed outside Provider to narrow types
5. **useSuspenseQuery alternative**: v5+ provides type-safe alternative
6. **Request waterfalls**: Context/Suspense can create waterfalls (stops rendering at Provider)
7. **Not state syncing**: Passing data via Context is like prop drilling, not copying state
8. **Use for mandatory data**: Best for data required for entire sub-tree (e.g., user info)
