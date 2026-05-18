# API Error Handling

APIs fail. Networks drop. Servers crash. If you do not handle errors, your app just... stops. And the user stares at a broken screen with no idea what happened. I have been that developer, and I have been that user. Let us fix it.

## Basic Error State

The simplest pattern is an error state alongside your loading and data states:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetch(`/api/users/${userId}`)
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((data) => { if (!cancelled) setUser(data); })
      .catch((err) => { if (!cancelled) setError(err.message); })
      .finally(() => { if (!cancelled) setLoading(false); });

    return () => { cancelled = true; };
  }, [userId]);

  if (loading) return <Spinner />;
  if (error) return <ErrorDisplay message={error} onRetry={refetch} />;
  return <UserCard user={user} />;
}
```

Always check `res.ok`. A 404 or 500 response does not throw an error with fetch. It returns a response with a bad status code. You have to check manually.

## User-Friendly Error Messages

Raw error messages like "HTTP 500" mean nothing to users. Map them to something helpful:

```jsx
function getErrorMessage(error) {
  if (error.message.includes("404")) return "We could not find that resource.";
  if (error.message.includes("403")) return "You do not have permission to view this.";
  if (error.message.includes("500")) return "Something went wrong on our end. Try again later.";
  if (error.message.includes("Failed to fetch")) return "Check your internet connection and try again.";
  return "Something went wrong. Please try again.";
}
```

## Retry Logic

Sometimes a request just fails because of a hiccup. Retrying automatically can save the day:

```jsx
function useFetchWithRetry(url, retries = 3) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;
    let attempt = 0;

    async function doFetch() {
      try {
        setLoading(true);
        const res = await fetch(url);
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const json = await res.json();
        if (!cancelled) { setData(json); setError(null); }
      } catch (err) {
        attempt++;
        if (attempt < retries && !cancelled) {
          const delay = Math.pow(2, attempt) * 1000; // exponential backoff
          setTimeout(doFetch, delay);
        } else if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled && (error || data)) setLoading(false);
      }
    }

    doFetch();
    return () => { cancelled = true; };
  }, [url]);

  return { data, error, loading };
}
```

That exponential backoff (1s, 2s, 4s) prevents hammering a struggling server.

## Error Boundaries

For errors that crash your component tree, use an Error Boundary:

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <p>Something went wrong.</p>;
    }
    return this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<ErrorPage />}>
  <App />
</ErrorBoundary>
```

Error boundaries catch render errors. They do not catch errors in event handlers or async code. For those, use try/catch. Between the two patterns, your users will always see something helpful instead of a white screen.
