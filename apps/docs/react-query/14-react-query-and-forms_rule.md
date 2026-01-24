---
description: "React Query and forms: simple approach with staleTime Infinity, separate states for collaborative forms, prevent double submit, invalidate after mutation"
alwaysApply: false
---

# React Query and Forms Rules

## Simple Approach: Copy Server State to Form

- **For simple forms where you're the only editor, copy server state to form state**
  - Use `defaultValue` for uncontrolled inputs
  - Split form into separate component if you need `defaultValues`

```typescript
// ✅ Simple approach
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
        <input
          {...register('firstName')}
          defaultValue={data.firstName}  // ✅ Use defaultValue
        />
      </form>
    )
  }
}

// ✅ Split form for defaultValues
function PersonForm({ person, onSubmit }) {
  const { register, handleSubmit } = useForm({ defaultValues: person })
  return <form onSubmit={handleSubmit(onSubmit)}>{/* ... */}</form>
}
```

## Disable Background Updates When Using Simple Approach

- **If you copy server state to form, disable background updates**

```typescript
// ✅ Opt out of background updates
const { data } = useQuery({
  queryKey: ['person', id],
  queryFn: () => fetchPerson(id),
  staleTime: Infinity,  // Prevent background refetches
})
```

- **Why**: If background updates happen, form state won't update anyway, so there's no point in refetching.

## Keep Background Updates On: Separate States

- **For collaborative forms or when you want to see updates, rigorously separate states**
  - Use controlled fields with `Controller`
  - Derive state: user changes (client) or server state

```typescript
function PersonDetail({ id }) {
  const { data } = useQuery({
    queryKey: ['person', id],
    queryFn: () => fetchPerson(id),
    // ✅ Keep background updates on
  })
  const { control, handleSubmit } = useForm()

  if (data) {
    return (
      <form onSubmit={handleSubmit(mutate)}>
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
      </form>
    )
  }
}
```

- **How it works**: If user changed field (`field.value` exists), show client state. If user didn't change field, fall back to server state. Background updates will update untouched fields automatically.

## Prevent Double Submit

- **Use `isLoading` from mutation to disable submit button**

```typescript
const { mutate, isLoading } = useMutation({
  mutationFn: updatePerson,
})

return (
  <form onSubmit={handleSubmit(mutate)}>
    <input type="submit" disabled={isLoading} />
  </form>
)
```

## Invalidate and Reset After Mutation

- **After successful mutation, invalidate queries and reset form**

```typescript
const { mutate } = useMutation({
  mutationFn: updatePerson,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['person', id] })
    reset()  // ✅ Reset form after successful mutation
  },
})
```

## Don't Do

- **Don't use `defaultValues` with undefined data**
  - `defaultValues` requires data to exist
  - Fix: Split form into separate component or use `defaultValue` for uncontrolled inputs

- **Don't keep background updates on without separating states**
  - Form state won't update with background refetches
  - Fix: Either disable background updates (`staleTime: Infinity`) or separate states rigorously

- **Don't use uncontrolled fields for separated states**
  - Requires controlled fields with `Controller`
  - Works best for shallow forms (nested objects are harder)
