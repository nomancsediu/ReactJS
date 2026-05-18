# Caching and Invalidation

React Query's caching system is its superpower. Understanding how it works helps you write faster, more efficient apps. I was confused by the terminology at first, so let me break it down simply.

## How Caching Works

When a query completes, React Query stores the result in its cache using the query key. If another component requests the same data, React Query returns the cached version instead of making a new request.

```jsx
// Both components share the same cached data
function UserList() {
  const { data } = useQuery({ queryKey: ["users"], queryFn: fetchUsers });
  // ...
}

function UserCount() {
  const { data } = useQuery({ queryKey: ["users"], queryFn: fetchUsers });
  return <p>{data?.length} users</p>;
}
```

Only one network request is made, even though two components use the same query.

## staleTime vs gcTime

These two settings control cache behavior, and I mixed them up constantly at first:

**staleTime** - How long data is considered "fresh." While fresh, React Query will not refetch unless you force it. Default is `0`, meaning data is immediately stale.

```jsx
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  staleTime: 5 * 60 * 1000,  // fresh for 5 minutes
});
```

**gcTime** (formerly cacheTime) - How long inactive data stays in memory before garbage collection. Default is 5 minutes.

```jsx
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  gcTime: 10 * 60 * 1000,  // kept in cache for 10 minutes after inactive
});
```

The key difference: staleTime is about freshness (should we refetch?), gcTime is about memory (should we delete the cache?).

## When Does Refetching Happen?

By default, React Query refetches stale data when:

- The component mounts
- The window regains focus
- The network reconnects
- A refetch interval is set

You can control each of these:

```jsx
useQuery({
  queryKey: ["users"],
  queryFn: fetchUsers,
  refetchOnWindowFocus: false,
  refetchOnReconnect: true,
  refetchInterval: 30000,  // refetch every 30 seconds
});
```

## Invalidating Queries

When you mutate data, you need to tell React Query that the old cached data is no longer valid. That is what `invalidateQueries` does:

```jsx
const queryClient = useQueryClient();

// Invalidate a specific query
queryClient.invalidateQueries({ queryKey: ["users"] });

// Invalidate all queries starting with "users"
queryClient.invalidateQueries({ queryKey: ["users"] });

// Invalidate everything
queryClient.invalidateQueries();
```

Invalidation marks the data as stale and triggers a refetch if any component is currently using that query.

## Selective Invalidation

You can be precise about what to invalidate:

```jsx
// Only invalidate the list, not individual user queries
queryClient.invalidateQueries({
  queryKey: ["users"],
  exact: true,
});

// Invalidate queries matching a pattern
queryClient.invalidateQueries({
  queryKey: ["users"],
  predicate: (query) => query.state.data?.length > 0,
});
```

Getting caching right makes your app feel instant. Users see cached data immediately while fresh data loads in the background.
