# Mastering Mutations in React Query - Best Practices

## Best Practices

### 1. Prefer Invalidation Over Direct Updates

Most of the time, **invalidation should be preferred** over direct cache updates. Direct updates require more frontend code and duplicate backend logic. Sorted lists are particularly hard to update directly.

```typescript
// ✅ Preferred: Invalidation
const useAddComment = (id) => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (newComment) =>
      axios.post(`/posts/${id}/comments`, newComment),
    onSuccess: () => {
      // ✅ Refetch the comments list
      queryClient.invalidateQueries({
        queryKey: ['posts', id, 'comments']
      })
    },
  })
}
```

**When to use direct updates**: Only when the mutation already returns everything you need and you don't want to refetch.

```typescript
// ✅ Direct update when mutation response has all data
const useUpdateTitle = (id) => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: (newTitle) =>
      axios
        .patch(`/posts/${id}`, { title: newTitle })
        .then((response) => response.data),
    onSuccess: (newPost) => {
      // ✅ Update detail view directly
      queryClient.setQueryData(['posts', id], newPost)
    },
  })
}
```

### 2. Use `mutate` Instead of `mutateAsync` (Almost Always)

**Almost always use `mutate`** instead of `mutateAsync`. You can still access data/errors via callbacks, and you don't have to worry about error handling.

```typescript
// ✅ Preferred: Using mutate with callbacks
const onSubmit = () => {
  myMutation.mutate(someData, {
    onSuccess: (data) => history.push(data.url),
  })
}

// 🚨 Works, but missing error handling
const onSubmit = async () => {
  const data = await myMutation.mutateAsync(someData)
  history.push(data.url)
}

// 😕 Verbose error handling required
const onSubmit = async () => {
  try {
    const data = await myMutation.mutateAsync(someData)
    history.push(data.url)
  } catch (error) {
    // do nothing
  }
}
```

**When to use `mutateAsync`**: Only when you really need the Promise, e.g., firing multiple mutations concurrently or dependent mutations.

### 3. Return Invalidation Promise to Keep Mutation in Loading State

If you want your mutation to stay in loading state while related queries update, return the result of `invalidateQueries`:

```typescript
{
  // ✅ Will wait for query invalidation to finish
  onSuccess: () => {
    return queryClient.invalidateQueries({
      queryKey: ['posts', id, 'comments'],
    })
  }
}

{
  // 🚀 Fire and forget - will not wait
  onSuccess: () => {
    queryClient.invalidateQueries({
      queryKey: ['posts', id, 'comments']
    })
  }
}
```

### 4. Separate Concerns in Callbacks

**Best practice**: Separate logic-related callbacks from UI-related callbacks:

- **In `useMutation` callbacks**: Do things that are absolutely necessary and logic-related (like query invalidation)
- **In `mutate` callbacks**: Do UI-related things like redirects or showing toast notifications

```typescript
const useUpdateTodo = () =>
  useMutation({
    mutationFn: updateTodo,
    // ✅ Always invalidate the todo list (logic)
    onSuccess: () => {
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list']
      })
    },
  })

// In the component
const updateTodo = useUpdateTodo()
updateTodo.mutate(
  { title: 'newTitle' },
  // ✅ Only redirect if we're still on the detail page (UI)
  { onSuccess: () => history.push('/todos') }
)
```

**Why**: Callbacks on `mutate` might not fire if the component unmounts before the mutation finishes. This separation keeps query logic in custom hooks (reusable) while UI actions stay in components.

### 5. Use Objects for Multiple Variables

Mutations only take one argument for variables. Use an object for multiple variables:

```typescript
// 🚨 Invalid syntax - will NOT work
const mutation = useMutation({
  mutationFn: (title, body) => updateTodo(title, body),
})
mutation.mutate('hello', 'world')

// ✅ Use an object for multiple variables
const mutation = useMutation({
  mutationFn: ({ title, body }) => updateTodo(title, body),
})
mutation.mutate({ title: 'hello', body: 'world' })
```

### 6. Be Careful with Optimistic Updates

**Not every mutation needs optimistic updates**. Use them only when:
- You're quite certain the update will go through
- Instant user feedback is actually required (e.g., toggle buttons)
- The rollback UX is acceptable

**Considerations:**
- If the todo you're adding needs an id, where do you get it from?
- If the list is sorted, will you insert the new entry at the right position?
- What if another user added something else - will positions switch after refetch?

**When NOT to use optimistic updates:**
- Forms in dialogs that close on submit
- Redirects from detail to list after update
- Complex transformations that are hard to mimic

Sometimes it's enough to disable the button and show a loading animation.

## Not To Do

### 1. Don't Use `mutateAsync` Unless You Really Need the Promise

`mutateAsync` requires manual error handling and is more verbose. Use `mutate` with callbacks instead.

### 2. Don't Pass Multiple Arguments to `mutationFn`

Mutations only accept one variable argument. Use an object for multiple values.

### 3. Don't Over-Use Optimistic Updates

Optimistic updates add complexity. Only use them when instant feedback is truly required and rollback UX is acceptable.

### 4. Don't Put UI Logic in `useMutation` Callbacks

Put UI-related actions (redirects, toasts) in `mutate` callbacks, not `useMutation` callbacks. This keeps custom hooks reusable and prevents UI actions from firing after unmount.

### 5. Don't Forget Callback Execution Order

Callbacks on `useMutation` fire **before** callbacks on `mutate`. Callbacks on `mutate` might not fire at all if the component unmounts before the mutation finishes.
