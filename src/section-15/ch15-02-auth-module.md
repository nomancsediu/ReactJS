# Auth Module

The auth module is the first real feature we build. It covers login, registration, token management, and route protection. Everything we learned in the authentication section comes together here.

## Login Page

I start with a clean login form that handles validation and error states:

```jsx
import { useState } from 'react';
import { useAuth } from '../context/AuthContext';
import { useNavigate, useLocation } from 'react-router-dom';

function LoginPage() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [loading, setLoading] = useState(false);
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from?.pathname || '/dashboard';

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');
    setLoading(true);

    try {
      await login(email, password);
      navigate(from, { replace: true });
    } catch (err) {
      setError(err.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="login-container">
      <h1>Sign In</h1>
      {error && <div className="error-banner">{error}</div>}
      <form onSubmit={handleSubmit}>
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
        </div>
        <div className="form-group">
          <label htmlFor="password">Password</label>
          <input
            id="password"
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
          />
        </div>
        <button type="submit" disabled={loading}>
          {loading ? 'Signing in...' : 'Sign In'}
        </button>
      </form>
      <p>
        Do not have an account? <a href="/register">Register</a>
      </p>
    </div>
  );
}
```

## Register Page

The register page is similar but with extra fields:

```jsx
function RegisterPage() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');
  const [error, setError] = useState('');
  const { register } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setError('');

    if (password !== confirmPassword) {
      setError('Passwords do not match');
      return;
    }

    try {
      await register(name, email, password);
      navigate('/dashboard');
    } catch (err) {
      setError(err.message || 'Registration failed');
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={name}
        onChange={(e) => setName(e.target.value)} placeholder="Name" required />
      <input type="email" value={email}
        onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
      <input type="password" value={password}
        onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
      <input type="password" value={confirmPassword}
        onChange={(e) => setConfirmPassword(e.target.value)} placeholder="Confirm password" required />
      {error && <div className="error-banner">{error}</div>}
      <button type="submit">Create Account</button>
    </form>
  );
}
```

## Auth Context

The auth context ties everything together with login, logout, register, and token management:

```jsx
const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const res = await apiClient.get('/auth/me');
      setUser(res.data.user);
      tokenManager.setToken(res.data.accessToken);
    } catch {
      setUser(null);
    } finally {
      setLoading(false);
    }
  };

  const login = async (email, password) => {
    const res = await apiClient.post('/auth/login', { email, password });
    setUser(res.data.user);
    tokenManager.setToken(res.data.accessToken);
  };

  const register = async (name, email, password) => {
    const res = await apiClient.post('/auth/register', { name, email, password });
    setUser(res.data.user);
    tokenManager.setToken(res.data.accessToken);
  };

  const logout = async () => {
    await apiClient.post('/auth/logout');
    setUser(null);
    tokenManager.clearToken();
  };

  return (
    <AuthContext.Provider value={{ user, loading, login, register, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export const useAuth = () => useContext(AuthContext);
```

## Protected Routes

Wrap dashboard routes with the guard:

```jsx
<Route path="/dashboard" element={
  <ProtectedRoute><DashboardLayout /></ProtectedRoute>
} />
```

With this module in place, users can register, log in, and access protected pages. The auth state persists across page reloads through the `/auth/me` check.
