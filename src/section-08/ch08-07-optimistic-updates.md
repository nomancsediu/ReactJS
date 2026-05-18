# Optimistic Updates

Optimistic updates are one of my favorite React Query features. The idea is simple: update the UI immediately when the user takes an action, before the server even responds. If the request fails, roll it back. It makes your app feel incredibly fast.

## The Concept

Normally, you wait for the server response before updating the UI. With optimistic updates, you assume the request will succeed and update the UI right away. If it fails, you revert. It is like being optimistic that things will work out.

## A Real-World Example: Todo Toggle

Let us build a todo app where toggling completion feels instant:

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function TodoItem({ todo }) {
  const queryClient = useQueryClient();

  const toggleMutation = useMutation({
    mutationFn: async () => {
      const response = await fetch(`/api/todos/${todo.id}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ completed: !todo.completed }),
      });
      if (!response.ok) throw new Error("Failed to toggle todo");
      return response.json();
    },

    onMutate: async () => {
      // Cancel any outgoing refetches
      await queryClient.cancelQueries({ queryKey: ["todos"] });

      // Snapshot the previous value
      const previousTodos = queryClient.getQueryData(["todos"]);

      // Optimistically update to the new value
      queryClient.setQueryData(["todos"], (old) =>
        old.map((t) =>
          t.id === todo.id ? { ...t, completed: !t.completed } : t
        )
      );

      // Return context with the snapshot
      return { previousTodos };
    },

    onError: (err, variables, context) => {
      // Roll back to the snapshot
      queryClient.setQueryData(["todos"], context.previousTodos);
    },

    onSettled: () => {
      // Always refetch after error or success
      queryClient.invalidateQueries({ queryKey: ["todos"] });
    },
  });

  return (
    <div>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => toggleMutation.mutate()}
      />
      <span>{todo.title}</span>
    </div>
  );
}
```

## How It Works Step by Step

1. **onMutate** fires before the request is sent. We save the current data, then update the cache optimistically.
2. **The request is sent** to the server in the background.
3. **onError** fires if the request fails. We restore the saved snapshot, rolling back the UI.
4. **onSettled** fires in both success and error cases. We invalidate the query to ensure the cache matches the server.

## The Return Context

The object returned from `onMutate` is passed to `onError` as the third argument. This is how we access the snapshot for rollback. It is a clean pattern that keeps the logic organized.

## When to Use Optimistic Updates

Optimistic updates work best for:

- Toggle actions (complete/incomplete, like/unlike)
- Reorder operations (drag and drop)
- Delete operations where you can remove the item immediately
- Small text edits

Avoid them for:

- Critical financial transactions
- Operations where the server response changes the data significantly
- Actions that require server-side validation before showing the result

Used well, optimistic updates make your app feel premium. The user takes an action and sees the result instantly. No waiting, no spinners.
