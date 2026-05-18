# Higher Order Components

Higher Order Components, or HOCs, sounded intimidating when I first read about them. But they are actually a simple idea: a function that takes a component and returns a new component with extra behavior.

## What Is an HOC?

```jsx
// An HOC is just a function
function withLogging(WrappedComponent) {
  return function EnhancedComponent(props) {
    useEffect(() => {
      console.log(`${WrappedComponent.name} mounted`);
    }, []);

    return <WrappedComponent {...props} />;
  };
}
```

You wrap your component with the HOC, and it gets the extra behavior for free.

## withAuth Example

One of the most common HOC patterns is protecting routes that require authentication:

```jsx
function withAuth(WrappedComponent) {
  return function AuthGuard(props) {
    const { user, loading } = useAuth();

    if (loading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;

    return <WrappedComponent {...props} user={user} />;
  };
}

// Usage
const ProtectedDashboard = withAuth(Dashboard);
const ProtectedSettings = withAuth(Settings);
```

Any component wrapped with `withAuth` automatically checks authentication before rendering.

## withLoading Example

Another common pattern is adding loading behavior:

```jsx
function withLoading(WrappedComponent, fetchData) {
  return function LoadingWrapper(props) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
      fetchData(props)
        .then(setData)
        .catch(setError)
        .finally(() => setLoading(false));
    }, []);

    if (loading) return <Spinner />;
    if (error) return <ErrorMessage error={error} />;

    return <WrappedComponent {...props} data={data} />;
  };
}

// Usage
const UserListWithLoading = withLoading(
  UserList,
  () => fetch('/api/users').then(r => r.json())
);
```

## When to Use HOCs

HOCs are useful when you want to:

- Add the same behavior to many components
- Keep components focused on their main job
- Cross-cut concerns like auth, logging, or analytics

## Things I Learned the Hard Way

Pass through props with `{...props}` so the wrapped component still receives its expected props:

```jsx
// Good: pass all props through
return <WrappedComponent {...props} extraProp={extraData} />;

// Bad: swallow props
return <WrappedComponent extraProp={extraData} />;
```

Also, give your HOC a display name for better debugging:

```jsx
function withAuth(WrappedComponent) {
  function AuthGuard(props) {
    // ...
  }
  AuthGuard.displayName = `withAuth(${WrappedComponent.displayName || WrappedComponent.name})`;
  return AuthGuard;
}
```

## HOCs vs Hooks

These days, I reach for custom hooks before HOCs. Hooks are simpler and avoid the extra wrapper component. But HOCs still have their place, especially when you need to wrap entire components or work with class components.
