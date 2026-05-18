# Protected Routes and Auth Guards

One of the first things I wanted to build after learning React Router was a protected route. You know, those pages that only logged-in users can see. If you are not logged in, you get redirected to the login page. It sounds simple, but there are a few details to get right.

## Building an Auth Guard Component

A protected route is just a wrapper component that checks if the user is authenticated before rendering the child route.

```jsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();
  const location = useLocation();

  // Show nothing while checking auth state
  if (loading) {
    return <div className="loading-spinner" />;
  }

  // Not authenticated? Redirect to login
  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}

export default ProtectedRoute;
```

The key detail here is saving the current location in the navigation state. This way, after the user logs in, we can redirect them back to where they wanted to go.

## Using Protected Routes

Wrap any route that requires authentication with the guard component:

```jsx
import { Routes, Route } from 'react-router-dom';
import ProtectedRoute from './components/ProtectedRoute';

function App() {
  return (
    <Routes>
      <Route path="/login" element={<LoginPage />} />
      <Route
        path="/dashboard"
        element={
          <ProtectedRoute>
            <Dashboard />
          </ProtectedRoute>
        }
      />
      <Route
        path="/settings"
        element={
          <ProtectedRoute>
            <Settings />
          </ProtectedRoute>
        }
      />
    </Routes>
  );
}
```

## Redirect After Login

When the user logs in successfully, redirect them to the page they originally requested:

```jsx
function LoginPage() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from?.pathname || '/dashboard';

  const handleLogin = async (email, password) => {
    await login(email, password);
    navigate(from, { replace: true });
  };

  return (
    <form onSubmit={handleLogin}>
      {/* form fields */}
    </form>
  );
}
```

## Persisting Auth State

When the user refreshes the page, React loses all in-memory state. That means the auth state too. To fix this, I check for an existing token on app startup:

```jsx
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check if user has a valid session on mount
    async function checkAuth() {
      try {
        const response = await fetch('/api/auth/me', {
          credentials: 'include',
        });
        if (response.ok) {
          const data = await response.json();
          setUser(data.user);
        }
      } catch {
        setUser(null);
      } finally {
        setLoading(false);
      }
    }
    checkAuth();
  }, []);

  return (
    <AuthContext.Provider value={{ user, loading }}>
      {children}
    </AuthContext.Provider>
  );
}
```

The `loading` state is crucial. Without it, the protected route might redirect to login before the auth check finishes. That caused me a frustrating flicker before I added it.
