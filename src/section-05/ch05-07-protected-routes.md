# Protected Routes

Not every page should be accessible to everyone. Some pages require login. Others require specific roles like admin or moderator. Protected routes let you control who sees what.

## The Auth Guard Component

The simplest protected route is a wrapper component that checks authentication:

```jsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();

  if (loading) return <Spinner />;

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  return children;
}
```

Wrap any route that needs protection:

```jsx
<Routes>
  <Route path="/login" element={<Login />} />
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
```

If the user is not logged in, they are redirected to `/login`. The `replace` prop prevents them from pressing back to reach the protected page.

## Saving the Intended Destination

When you redirect to login, it is nice to send the user back where they wanted to go:

```jsx
function ProtectedRoute({ children }) {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) return <Spinner />;

  if (!user) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return children;
}
```

Then after login, navigate to the saved location:

```jsx
function Login() {
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from?.pathname || '/';

  const handleLogin = async (credentials) => {
    await login(credentials);
    navigate(from, { replace: true });
  };

  return <LoginForm onSubmit={handleLogin} />;
}
```

## Role-Based Access

For pages that need specific roles, make the guard configurable:

```jsx
function RoleRoute({ children, allowedRoles }) {
  const { user } = useAuth();

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  if (!allowedRoles.includes(user.role)) {
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

// Usage
<RoleRoute allowedRoles={['admin']}>
  <AdminPanel />
</RoleRoute>

<RoleRoute allowedRoles={['admin', 'moderator']}>
  <ModerationQueue />
</RoleRoute>
```

## A Layout-Based Approach

For apps with many protected routes, wrap them in a layout:

```jsx
<Routes>
  <Route path="/login" element={<Login />} />

  <Route
    element={
      <ProtectedRoute>
        <AppLayout />
      </ProtectedRoute>
    }
  >
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/settings" element={<Settings />} />
    <Route path="/profile" element={<Profile />} />
  </Route>
</Routes>
```

All child routes are automatically protected because the parent layout is wrapped. No need to protect each route individually.

## Key Takeaway

Protected routes are just components that check a condition before rendering their children. If the condition fails, redirect. It is simple, composable, and works with any auth system.
