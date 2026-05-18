# SWR

SWR is an alternative data fetching library created by Vercel, the same team behind Next.js. The name comes from stale-while-revalidate, a caching strategy. It is simpler than React Query but still powerful.

## What Is SWR?

SWR follows a specific strategy: return cached data first (stale), then fetch new data in the background (revalidate), and finally update the UI with the fresh data. This gives users instant feedback while keeping data current.

## Installing SWR

```bash
npm install swr
```

## Basic Usage

```jsx
import useSWR from "swr";

const fetcher = (url) => fetch(url).then((res) => res.json());

function UserList() {
  const { data, error, isLoading } = useSWR("/api/users", fetcher);

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

The first argument is the key (usually a URL). The second is the fetcher function. If the key is `null`, SWR skips the request. This is handy for conditional fetching:

```jsx
const { data } = useSWR(userId ? `/api/users/${userId}` : null, fetcher);
```

## Configuration

You can configure SWR globally or per hook:

```jsx
// Global config
import { SWRConfig } from "swr";

function App() {
  return (
    <SWRConfig
      value={{
        fetcher: (url) => fetch(url).then((r) => r.json()),
        revalidateOnFocus: false,
        refreshInterval: 30000,
      }}
    >
      <YourApp />
    </SWRConfig>
  );
}
```

## Mutation

SWR has a `mutate` function for updating data:

```jsx
import useSWR, { useSWRConfig } from "swr";

function CreateUser() {
  const { mutate } = useSWRConfig();

  const handleCreate = async () => {
    await fetch("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ name: "New User" }),
    });

    // Tell SWR to refetch the users list
    mutate("/api/users");
  };

  return <button onClick={handleCreate}>Create User</button>;
}
```

## SWR vs React Query

| Feature | SWR | React Query |
|---------|-----|-------------|
| Bundle size | Smaller | Larger |
| DevTools | Limited | Full DevTools |
| Mutations | Basic | First-class |
| Pagination | Manual | Built-in |
| Infinite scroll | With plugin | Built-in |
| Optimistic updates | Manual | Built-in |
| Learning curve | Lower | Higher |
| Community | Smaller | Larger |

I reach for SWR when I want something lightweight and my app has mostly read operations. For apps with complex mutations, pagination, or offline support, React Query is the better choice. Both are excellent libraries. Pick the one that fits your needs.
