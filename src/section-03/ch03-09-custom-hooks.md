# Custom Hooks

Custom hooks let you extract component logic into reusable functions. Once you start seeing the same patterns repeated across components, it is time to create a custom hook. They are one of my favorite React features because they make code so much cleaner.

## Naming Convention

Custom hooks always start with `use`. This is not just a convention. The React linting rules rely on it to enforce the rules of hooks:

```jsx
// Custom hook
function useWindowSize() { ... }

// Not a custom hook (React will not enforce hook rules)
function getWindowSize() { ... }
```

## Example: useFetch

Fetching data is something you do in many components. Let me turn the common pattern into a reusable hook:

```jsx
function useFetch(url) {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)

  useEffect(() => {
    let cancelled = false

    async function fetchData() {
      try {
        setLoading(true)
        setError(null)
        const response = await fetch(url)
        if (!response.ok) throw new Error(`HTTP ${response.status}`)
        const json = await response.json()
        if (!cancelled) setData(json)
      } catch (err) {
        if (!cancelled) setError(err.message)
      } finally {
        if (!cancelled) setLoading(false)
      }
    }

    fetchData()
    return () => { cancelled = true }
  }, [url])

  return { data, loading, error }
}
```

Using it:

```jsx
function UserList() {
  const { data: users, loading, error } = useFetch("/api/users")

  if (loading) return <p>Loading...</p>
  if (error) return <p>Error: {error}</p>
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  )
}
```

All the fetching logic in one place, reusable everywhere.

## Example: useLocalStorage

Persisting state to localStorage is another common pattern:

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const stored = localStorage.getItem(key)
      return stored ? JSON.parse(stored) : initialValue
    } catch {
      return initialValue
    }
  })

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value))
  }, [key, value])

  return [value, setValue]
}
```

Using it:

```jsx
function Settings() {
  const [username, setUsername] = useLocalStorage("username", "")
  return (
    <input
      value={username}
      onChange={(e) => setUsername(e.target.value)}
    />
  )
}
```

The value persists across page refreshes automatically.

## Example: useDebounce

Debouncing prevents a function from running too frequently. Useful for search inputs:

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value)

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(timer)
  }, [value, delay])

  return debouncedValue
}
```

Using it:

```jsx
function Search() {
  const [query, setQuery] = useState("")
  const debouncedQuery = useDebounce(query, 300)

  useEffect(() => {
    if (debouncedQuery) {
      fetchResults(debouncedQuery)
    }
  }, [debouncedQuery])

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />
}
```

## Tips for Writing Custom Hooks

- Call other hooks at the top level of your custom hook
- Return an object or array from your hook, just like built-in hooks
- Keep hooks focused on one responsibility
- If two pieces of logic are unrelated, make two separate hooks
- Add the `use` prefix. It is required for the lint rules to work

Custom hooks are where React really shines. They let you build abstractions that feel like they are part of the framework itself.
