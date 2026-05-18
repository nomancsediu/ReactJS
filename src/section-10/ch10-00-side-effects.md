# Real-World API Integration

So far we have built components, managed state, and organized our apps. But a real app needs to talk to the outside world. It needs to fetch data from APIs, send forms, upload files, and handle responses that take time.

This is where side effects come in. Everything we did before was pure: input goes in, UI comes out. Now we are stepping into the messy, unpredictable world of networks, servers, and async operations.

I remember the first time I tried to fetch data in a React component. I put `fetch()` right in the render function and wondered why my app made a hundred requests. It was a mess. That is exactly the kind of mistake this section will help you avoid.

## What You Will Learn

In this section, we cover the practical side of working with APIs in React:

- **What side effects are** and why they need special handling in React
- **CRUD operations** for creating, reading, updating, and deleting data
- **Loading states** so your users are not staring at blank screens
- **Error handling** because APIs fail, networks drop, and things go wrong
- **Pagination and infinite scroll** for dealing with large datasets
- **Real-time data** with WebSockets and polling for live updates
- **File uploads** with progress tracking, previews, and drag and drop

## The Big Picture

Here is the thing about side effects: they break the pure function model that makes React predictable. When you fetch data, you are reaching outside React's world. That means you need to tell React exactly when and how to do it.

```jsx
// This is what we are building toward
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        if (!response.ok) throw new Error("Failed to fetch");
        const data = await response.json();
        if (!cancelled) setUser(data);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    }

    fetchUser();
    return () => { cancelled = true; };
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  return <UserCard user={user} />;
}
```

That pattern above is the foundation. Every chapter in this section builds on it. By the end, you will handle real API calls with confidence, loading states, errors, and all the messy stuff that makes a real app work.

Let us dive in.
