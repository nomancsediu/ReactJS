# Data Fetching

Getting data from APIs is a core part of almost every React app. Whether you are loading a list of products, fetching user profiles, or posting form data to a server, you need a solid understanding of how to communicate with backends.

## The Challenge

Data fetching in React is not just about making a request. You have to handle several concerns:

- **Loading states** so the user knows something is happening
- **Error handling** when the request fails
- **Caching** to avoid redundant requests
- **Refreshing** to keep data up to date
- **Race conditions** when multiple requests fire at once

When I first started, I just put `fetch` calls inside `useEffect` and called it a day. It worked for simple cases, but things got complicated fast. What if the component unmounts before the request finishes? What if the user navigates away and back? What if two components need the same data?

## What We Will Cover

This section walks through the evolution of data fetching in React:

1. **Fetch API** - The native browser API for HTTP requests
2. **Axios** - A popular HTTP client with extra features
3. **Custom API hooks** - Wrapping fetch logic into reusable hooks
4. **React Query setup** - Setting up TanStack Query in your project
5. **Queries and mutations** - Reading and writing data with React Query
6. **Caching and invalidation** - How React Query manages your data
7. **Optimistic updates** - Updating the UI before the server responds
8. **SWR** - An alternative to React Query

My goal is to help you understand the problem space first, then show you the tools that solve it. That way, when you reach for a library, you know exactly why you need it.
