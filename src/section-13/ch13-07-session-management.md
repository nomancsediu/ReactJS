# Session Management

Getting the user logged in is only half the battle. Keeping their session alive and handling it properly when it ends is just as important. I learned this after building an app where users kept getting randomly logged out. Not a great experience.

## Session Persistence

When a user closes the browser and comes back later, they should still be logged in if they chose to. This is session persistence. There are two common approaches:

**Option 1: Remember Me checkbox**

```jsx
function LoginForm() {
  const [rememberMe, setRememberMe] = useState(false);

  const handleLogin = async (email, password) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password, rememberMe }),
    });

    const data = await response.json();

    if (rememberMe) {
      localStorage.setItem('rememberMe', 'true');
    } else {
      sessionStorage.setItem('sessionOnly', 'true');
    }
  };

  return (
    <form>
      <input type="email" placeholder="Email" />
      <input type="password" placeholder="Password" />
      <label>
        <input
          type="checkbox"
          checked={rememberMe}
          onChange={(e) => setRememberMe(e.target.checked)}
        />
        Remember me
      </label>
      <button type="submit">Log In</button>
    </form>
  );
}
```

**Option 2: Let the backend handle it**

If you use HttpOnly cookies for refresh tokens, the server can set the cookie expiry based on whether the user wants to be remembered. A "remember me" session might last 30 days, while a regular session might last only 24 hours.

## Logout

Logout is more than just clearing the frontend state. You need to invalidate the session on the server too.

```jsx
const logout = async () => {
  try {
    // Tell the server to invalidate the refresh token
    await fetch('/api/auth/logout', {
      method: 'POST',
      credentials: 'include',
    });
  } catch {
    // Even if the server call fails, clear local state
  } finally {
    setUser(null);
    tokenManager.clearToken();
    localStorage.removeItem('rememberMe');
    navigate('/login');
  }
};
```

I always put the cleanup in a `finally` block. Even if the server call fails (maybe the network is down), the user should still be logged out locally.

## Session Timeout

For security, you might want to log users out after a period of inactivity. I use a simple timer approach:

```jsx
function useSessionTimeout(timeoutMinutes = 30) {
  const { logout } = useAuth();
  const timerRef = useRef(null);

  const resetTimer = useCallback(() => {
    clearTimeout(timerRef.current);
    timerRef.current = setTimeout(() => {
      logout();
      alert('Your session has expired due to inactivity.');
    }, timeoutMinutes * 60 * 1000);
  }, [logout, timeoutMinutes]);

  useEffect(() => {
    const events = ['mousedown', 'keydown', 'scroll', 'touchstart'];

    events.forEach((event) => {
      window.addEventListener(event, resetTimer);
    });

    resetTimer();

    return () => {
      clearTimeout(timerRef.current);
      events.forEach((event) => {
        window.removeEventListener(event, resetTimer);
      });
    };
  }, [resetTimer]);
}
```

This hook resets a timer on every user interaction. If the user is idle for too long, it automatically logs them out. You can adjust the timeout based on your app's security needs.

The biggest lesson I learned: always handle the cleanup. A broken logout is worse than no logout, because it gives users a false sense of security.
