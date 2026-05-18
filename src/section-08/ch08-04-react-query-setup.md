# React Query Setup

TanStack Query (formerly React Query) is the most popular data fetching library for React. It handles caching, background refetching, deduplication, and so much more. Setting it up is straightforward, but understanding the pieces helps.

## Why React Query?

My custom hooks worked fine, but they had limitations. No caching between components, no background refetching, no deduplication of identical requests. React Query solves all of this and more with very little setup.

## Installing

```bash
npm install @tanstack/react-query
```

## QueryClient and QueryClientProvider

React Query needs a `QueryClient` instance and a provider at the root of your app:

```jsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,  // 5 minutes
      retry: 1,
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  );
}
```

The `QueryClient` is the central manager. It stores all cached data and handles queries. The `QueryClientProvider` makes it available to all components.

## QueryClient Options

You can configure global defaults:

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60,       // data is fresh for 1 minute
      gcTime: 1000 * 60 * 5,      // cache is kept for 5 minutes (formerly cacheTime)
      retry: 2,                     // retry failed requests twice
      refetchOnWindowFocus: false,  // do not refetch when tab regains focus
    },
  },
});
```

## DevTools

React Query comes with DevTools that are incredibly helpful for debugging. They show you cached queries, their status, and when they will refetch:

```bash
npm install @tanstack/react-query-devtools
```

```jsx
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

The DevTools only render in development, so they do not add to your production bundle size. I keep them open almost always while developing. They show you exactly what queries are running, what is cached, and what is stale.

## Next Steps

With the setup done, we can start using `useQuery` and `useMutation` to fetch and modify data. That is what the next chapter is all about.
