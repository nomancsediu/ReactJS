# 404 Pages and Error Boundaries

Users will visit URLs that do not exist. API calls will fail. Components will throw errors. How you handle these moments makes or breaks the user experience.

## Catch-All Route for 404

The simplest way to handle unknown URLs is a catch-all route at the bottom of your route list:

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  <Route path="/blog" element={<Blog />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

The `*` path matches anything that was not matched by earlier routes.

```jsx
function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-6xl font-bold text-gray-300">404</h1>
      <p className="text-xl mt-4">Page not found</p>
      <Link to="/" className="mt-6 text-blue-500 underline">
        Go home
      </Link>
    </div>
  );
}
```

The `*` route must be last. React Router checks routes in order of specificity, but the wildcard catches everything, so place it at the end.

## Error Elements in React Router

React Router v6.4 added built-in error handling with the `errorElement` prop:

```jsx
<Route
  path="/users/:id"
  element={<UserProfile />}
  errorElement={<UserError />}
/>
```

If `UserProfile` throws during rendering, `UserError` renders instead. The error is available through `useRouteError`:

```jsx
import { useRouteError } from 'react-router-dom';

function UserError() {
  const error = useRouteError();

  return (
    <div>
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <Link to="/">Go home</Link>
    </div>
  );
}
```

## Global Error Boundary

For errors that happen outside of React Router, use a class-based error boundary:

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, info) {
    console.error('Error caught:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="p-8">
          <h1>Something broke</h1>
          <p>{this.state.error.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

Wrap your app with it:

```jsx
<ErrorBoundary>
  <BrowserRouter>
    <App />
  </BrowserRouter>
</ErrorBoundary>
```

## A Complete Setup

Putting it all together:

```jsx
function App() {
  return (
    <ErrorBoundary>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route
          path="/users/:id"
          element={<UserProfile />}
          errorElement={<UserError />}
        />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </ErrorBoundary>
  );
}
```

- Unknown URLs hit the 404 page
- Route-specific errors show contextual error UI
- Unexpected errors are caught by the error boundary

Handling errors gracefully is one of those things users do not notice when it works well, but they definitely notice when it does not.
