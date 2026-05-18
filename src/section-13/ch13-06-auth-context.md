# Auth Context and useAuth Hook

Passing auth state through props is painful. I tried it once and ended up threading `user` and `onLogout` through five levels of components. That is when I learned about React Context. An auth context gives every component access to the current user and auth functions without prop drilling.

## Creating the Auth Context

Here is how I structure my auth context:

```jsx
import { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check for existing session on mount
    async function initializeAuth() {
      try {
        const response = await fetch('/api/auth/me', {
          credentials: 'include',
        });
        if (response.ok) {
          const data = await response.json();
          setUser(data.user);
          tokenManager.setToken(data.accessToken);
        }
      } catch {
        setUser(null);
      } finally {
        setLoading(false);
      }
    }
    initializeAuth();
  }, []);

  const login = async (email, password) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({ email, password }),
    });

    if (!response.ok) throw new Error('Login failed');

    const data = await response.json();
    setUser(data.user);
    tokenManager.setToken(data.accessToken);
  };

  const logout = async () => {
    await fetch('/api/auth/logout', {
      method: 'POST',
      credentials: 'include',
    });
    setUser(null);
    tokenManager.clearToken();
  };

  const value = {
    user,
    loading,
    login,
    logout,
    isAuthenticated: !!user,
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}
```

## The useAuth Hook

The context is not very useful without a convenient hook. I always create a `useAuth` hook:

```jsx
export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```

That error message is important. I spent hours debugging why `useAuth` returned undefined before I added it. If you forget to wrap your app with `AuthProvider`, this error tells you exactly what went wrong.

## Using the Hook in Components

Now any component can access auth state and functions:

```jsx
import { useAuth } from '../hooks/useAuth';

function Header() {
  const { user, logout, isAuthenticated } = useAuth();

  return (
    <header>
      <h1>My App</h1>
      {isAuthenticated ? (
        <div>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Log Out</button>
        </div>
      ) : (
        <a href="/login">Log In</a>
      )}
    </header>
  );
}

function ProfilePage() {
  const { user, loading } = useAuth();

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

## Wrapping the App

Do not forget to wrap your entire app with the provider, typically in your root file:

```jsx
import { AuthProvider } from './context/AuthProvider';
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          {/* your routes */}
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

This pattern keeps auth logic centralized, reusable, and testable. I never go back to prop drilling after using it.
