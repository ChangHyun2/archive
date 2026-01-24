# React Query and Forms - Best Practices

## Key Concept

Forms blur the line between **Server State** and **Client State**. When displaying server data in a form for editing, you need to decide how to handle this hybrid state.

## Best Practices

### 1. Simple Approach: Copy Server State to Form (With Tradeoffs)

For simple forms where you're the only editor, copy server state to form state:

```typescript
function PersonDetail({ id }) {
  const { data } = useQuery({
    queryKey: ['person', id],
    queryFn: () => fetchPerson(id),
  })
  const { register, handleSubmit } = useForm()
  const { mutate } = useMutation({
    mutationFn: (values) => updatePerson(values),
  })

  if (data) {
    return (
      <form onSubmit={handleSubmit(mutate)}>
        <div>
          <label htmlFor="firstName">First Name</label>
          <input
            {...register('firstName')}
            defaultValue={data.firstName}  // ✅ Use defaultValue
          />
        </div>
        <input type="submit" />
      </form>
    )
  }

  return 'loading...'
}
```

**Tradeoffs**:
- ✅ Simple and works well
- ❌ No background updates (form won't update if server data changes)
- ❌ Can't use `defaultValues` directly (data is undefined on first render)

**Solution for defaultValues**: Split form into separate component:

```typescript
function PersonDetail({ id }) {
  const { data } = useQuery({
    queryKey: ['person', id],
    queryFn: () => fetchPerson(id),
  })
  const { mutate } = useMutation({
    mutationFn: (values) => updatePerson(values),
  })

  if (data) {
    return <PersonForm person={data} onSubmit={mutate} />
  }

  return 'loading...'
}

function PersonForm({ person, onSubmit }) {
  // ✅ Now data is guaranteed to exist
  const { register, handleSubmit } = useForm({ defaultValues: person })
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* ... */}
    </form>
  )
}
```

### 2. Disable Background Updates When Using Simple Approach

If you copy server state to form, disable background updates to avoid confusion:

```typescript
// ✅ Opt out of background updates
const { data } = useQuery({
  queryKey: ['person', id],
  queryFn: () => fetchPerson(id),
  staleTime: Infinity,  // Prevent background refetches
})
```

**Why**: If background updates happen, form state won't update anyway, so there's no point in refetching.

### 3. Keep Background Updates On: Separate States

For collaborative forms or when you want to see updates, rigorously separate states:

```typescript
function PersonDetail({ id }) {
  const { data } = useQuery({
    queryKey: ['person', id],
    queryFn: () => fetchPerson(id),
    // ✅ Keep background updates on
  })
  const { control, handleSubmit } = useForm()
  const { mutate } = useMutation({
    mutationFn: (values) => updatePerson(values),
  })

  if (data) {
    return (
      <form onSubmit={handleSubmit(mutate)}>
        <div>
          <label htmlFor="firstName">First Name</label>
          <Controller
            name="firstName"
            control={control}
            render={({ field }) => (
              // ✅ Derive state: user changes (client) or server state
              <input
                {...field}
                value={field.value ?? data.firstName}
              />
            )}
          />
        </div>
        <input type="submit" />
      </form>
    )
  }

  return 'loading...'
}
```

**How it works**:
- If user changed field (`field.value` exists), show client state
- If user didn't change field, fall back to server state (`data.firstName`)
- Background updates will update untouched fields automatically

**Caveats**:
- Requires controlled fields (can't use uncontrolled)
- Works best for shallow forms (nested objects are harder)
- Might be confusing UX to change form values in background

**Update**: React Hook Form has a new `values` API that reacts to changes and updates form values. Use this instead of `defaultValues` to derive state from server state.

### 4. Prevent Double Submit

Use `isLoading` from mutation to disable submit button:

```typescript
const { mutate, isLoading } = useMutation({
  mutationFn: (values) => updatePerson(values)
})

<input type="submit" disabled={isLoading} />
```

### 5. Invalidate and Reset After Mutation

If you don't redirect after submission, reset form after invalidation completes:

```typescript
function PersonDetail({ id }) {
  const queryClient = useQueryClient()
  const { data } = useQuery({
    queryKey: ['person', id],
    queryFn: () => fetchPerson(id),
  })
  const { control, handleSubmit, reset } = useForm()
  const { mutate } = useMutation({
    mutationFn: updatePerson,
    // ✅ Return Promise from invalidation so it's awaited
    onSuccess: () =>
      queryClient.invalidateQueries({ queryKey: ['person', id] }),
  })

  if (data) {
    return (
      <form
        onSubmit={handleSubmit((values) =>
          // ✅ Reset client state back to undefined
          mutate(values, { onSuccess: () => reset() })
        )}
      >
        {/* ... */}
      </form>
    )
  }

  return 'loading...'
}
```

**Why**: With separated states, resetting to `undefined` makes form fall back to server state, which will be updated after invalidation.

## Not To Do

### 1. Don't Use defaultValues with Undefined Data

```typescript
// 🚨 This will initialize form with undefined
const { data } = useQuery({
  queryKey: ['person', id],
  queryFn: () => fetchPerson(id),
})
const { register } = useForm({ defaultValues: data })  // ❌ data is undefined!
```

**Solution**: Split form component or use `defaultValue` on individual fields.

### 2. Don't Keep Background Updates On Without Separating States

If you copy server state to form state, you must disable background updates (`staleTime: Infinity`). Otherwise, you'll refetch data that won't be reflected in the form anyway.

### 3. Don't Use Uncontrolled Fields for Separated States

The separated states approach requires controlled fields. Uncontrolled fields won't work because you need to derive value from both client and server state.

### 4. Don't Forget to Handle Collaborative Editing

For forms where multiple people can edit:
- Use separated states approach
- Consider highlighting fields that are out of sync
- Let users decide what to do with conflicting changes

## Key Insights

1. **Forms blur Server/Client state**: Decide whether to copy state or keep them separate
2. **Simple approach**: Copy to form + `staleTime: Infinity` (no background updates)
3. **Advanced approach**: Separate states + derive value (supports background updates)
4. **Always prevent double submit**: Use `isLoading` to disable submit button
5. **Reset after mutation**: Reset form after invalidation completes (with separated states)
