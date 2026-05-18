# Custom API Hooks

After using fetch and Axios directly in components, I realized I was writing the same loading, error, and data logic over and over. The solution? Custom hooks that encapsulate the entire data fetching pattern.

## The Loading/Error/Data Pattern

Every data fetch has three states: loading, error, and success. A custom hook can manage all three:

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        const result = await response.json();
        if (!cancelled) setData(result);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    };

    fetchData();

    return () => { cancelled = true; };
  }, [url]);

  return { data, loading, error };
}
```

That `cancelled` flag prevents state updates if the component unmounts before the request finishes. I learned this the hard way after seeing "Cannot update state on unmounted component" warnings.

## Using the Hook

```jsx
function UserList() {
  const { data: users, loading, error } = useFetch("/api/users");

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## useMutation Hook

For POST, PUT, and DELETE operations, I use a mutation hook:

```jsx
function useMutation(url, method = "POST") {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const mutate = async (body) => {
    setLoading(true);
    setError(null);

    try {
      const response = await fetch(url, {
        method,
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      });
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const result = await response.json();
      return result;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };

  return { mutate, loading, error };
}
```

```jsx
function CreateUser() {
  const { mutate, loading } = useMutation("/api/users");

  const handleSubmit = async (e) => {
    e.preventDefault();
    await mutate({ name: "New User", email: "new@test.com" });
  };

  return (
    <button onClick={handleSubmit} disabled={loading}>
      {loading ? "Creating..." : "Create User"}
    </button>
  );
}
```

Custom hooks are a great middle ground between raw fetch and a full library. They work well for small to medium apps.
