---
description: "React Query and React Context: use Context as dependency injection tool, add invariants for type safety, use useSuspenseQuery (v5+)"
alwaysApply: false
---

# React Query and React Context Rules

## Use React Context as Dependency Injection Tool

- **React Context is a dependency injection tool**, not a state manager
  - Use it to make implicit dependencies explicit when you know data exists but TypeScript can't verify it

## Use React Context for Explicit Dependencies

- **Make implicit dependencies explicit with Context**

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

- **Benefits**:
  - Makes dependency explicit (visible in code)
  - Type-safe access
  - Clear contract: Provider must exist
  - Refactoring-safe (moving Provider affects all consumers)

## Add Invariant for Type Safety

- **Throw error if context accessed outside Provider**

```typescript
export const useCurrentUserContext = () => {
  const context = React.useContext(CurrentUserContext)
  
  if (!context) {
    throw new Error('useCurrentUserContext must be used within CurrentUserContextProvider')
  }
  
  return context  // ✅ TypeScript knows context is not null
}
```

## Use useSuspenseQuery (v5+)

- **v5+ provides `useSuspenseQuery` which makes data non-nullable**

```typescript
// ✅ v5+ - data is guaranteed to exist
export const CurrentUserContextProvider = ({ children }) => {
  const { data } = useSuspenseQuery({
    queryKey: ['currentUser'],
    queryFn: fetchCurrentUser,
  })

  return (
    <CurrentUserContext.Provider value={data}>  // ✅ data is User, not User | undefined
      {children}
    </CurrentUserContext.Provider>
  )
}
```

## Don't Do

- **Don't use Context for state management**
  - React Context is for dependency injection, not state management
  - Fix: Use React Query for server state, other tools for client state

- **Don't use type assertions**
  - Unsafe and error-prone
  - Fix: Add invariants or use `useSuspenseQuery` (v5+)
