# Queries and Mutations

Now that React Query is set up, let us use it. Queries fetch data. Mutations modify data. Together they cover everything you need for API communication.

## useQuery for Fetching Data

`useQuery` takes a query key and a query function:

```jsx
import { useQuery } from "@tanstack/react-query";

function UserList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ["users"],
    queryFn: async () => {
      const response = await fetch("/api/users");
      if (!response.ok) throw new Error("Failed to fetch users");
      return response.json();
    },
  });

  if (isLoading) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## Query Keys

Query keys are how React Query identifies and caches queries. They can be simple strings or complex arrays:

```jsx
// Simple key
useQuery({ queryKey: ["users"], queryFn: fetchUsers });

// Key with variables
useQuery({ queryKey: ["users", userId], queryFn: () => fetchUser(userId) });

// Key with filters
useQuery({ queryKey: ["users", { status: "active" }], queryFn: fetchActiveUsers });
```

Keys are compared deeply. `["users", 1]` and `["users", 2]` are different queries with separate caches. This is powerful for caching individual items.

## useMutation for Modifying Data

`useMutation` handles POST, PUT, DELETE, and any operation that changes data:

```jsx
import { useMutation } from "@tanstack/react-query";

function CreateUser() {
  const mutation = useMutation({
    mutationFn: async (newUser) => {
      const response = await fetch("/api/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(newUser),
      });
      if (!response.ok) throw new Error("Failed to create user");
      return response.json();
    },
    onSuccess: () => {
      console.log("User created successfully!");
    },
    onError: (error) => {
      console.error("Error:", error.message);
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ name: "Alice", email: "alice@test.com" })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? "Creating..." : "Create User"}
    </button>
  );
}
```

## Variables

You pass variables to `mutate()`:

```jsx
const mutation = useMutation({
  mutationFn: async ({ id, data }) => {
    const response = await fetch(`/api/users/${id}`, {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });
    return response.json();
  },
});

// Call with variables
mutation.mutate({ id: 1, data: { name: "Updated Name" } });
```

## Invalidating Queries After Mutations

After creating or updating data, you usually want to refetch the list. Use `queryClient.invalidateQueries` in `onSuccess`:

```jsx
import { useQueryClient } from "@tanstack/react-query";

const mutation = useMutation({
  mutationFn: createUser,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ["users"] });
  },
});
```

This tells React Query that the "users" data is stale and needs refetching. It is the standard pattern for keeping your UI in sync with the server.
