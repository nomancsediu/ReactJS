# Context API

The Context API is React's built-in solution for sharing state across the component tree without prop drilling. I avoided it at first because it seemed complicated, but it is actually straightforward.

## Creating Context

First, create a context with `createContext`:

```jsx
import { createContext, useState } from 'react';

const ThemeContext = createContext();
```

The context itself does not hold state. It is a container for passing data through the tree.

## The Provider Component

Wrap the components that need the data with a Provider. The Provider accepts a `value` prop:

```jsx
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

The Provider holds the state and makes it available to everything inside it.

## Consuming Context

Use the `useContext` hook to read context in any descendant:

```jsx
import { useContext } from 'react';

function Header() {
  const { theme, toggleTheme } = useContext(ThemeContext);

  return (
    <header className={theme === 'dark' ? 'bg-gray-900 text-white' : 'bg-white'}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

function Footer() {
  const { theme } = useContext(ThemeContext);

  return (
    <footer className={theme === 'dark' ? 'bg-gray-800 text-gray-200' : 'bg-gray-100'}>
      Built with React
    </footer>
  );
}
```

Both `Header` and `Footer` access the theme without a single prop being passed.

## Wrapping Your App

Put the Provider high enough in the tree that all consumers are inside it:

```jsx
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
      <Footer />
    </ThemeProvider>
  );
}
```

## Multiple Contexts

You can have as many contexts as you need. Just nest the providers:

```jsx
function App() {
  return (
    <ThemeProvider>
      <AuthProvider>
        <LocaleProvider>
          <Router>
            <Layout />
          </Router>
        </LocaleProvider>
      </AuthProvider>
    </ThemeProvider>
  );
}
```

Each consumer only subscribes to the context it uses, so they do not interfere with each other.

## A Practical Auth Context

Here is a pattern I use in most projects:

```jsx
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const savedUser = localStorage.getItem('user');
    if (savedUser) setUser(JSON.parse(savedUser));
    setLoading(false);
  }, []);

  const login = async (credentials) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
    });
    const userData = await response.json();
    setUser(userData);
    localStorage.setItem('user', JSON.stringify(userData));
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}
```

Creating a custom hook like `useAuth` makes consumption cleaner and gives you one place to add error handling if the context is used outside its provider.

Context is not a replacement for every state library. It is perfect for app-wide data like auth, theme, and locale. For more complex scenarios, you might need additional tools.
