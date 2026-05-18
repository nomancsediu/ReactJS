# useContext

Prop drilling is when you pass data through multiple layers of components that do not need it themselves, just to get it to a deeply nested child. `useContext` solves this by letting any component read a value directly, no matter how deep it is in the tree.

## The Problem

```jsx
function App() {
  const [theme, setTheme] = useState("dark")
  return <Layout theme={theme} />  // pass theme down
}

function Layout({ theme }) {
  return <Sidebar theme={theme} />  // Layout does not use theme, just passes it
}

function Sidebar({ theme }) {
  return <Button theme={theme} />   // Sidebar does not use theme either
}

function Button({ theme }) {
  return <button className={`btn-${theme}`}>Click</button>  // finally used
}
```

Three levels of passing `theme` just to get it to `Button`. Imagine this with ten levels and five different props. It gets messy fast.

## Creating Context

```jsx
import { createContext } from "react"

const ThemeContext = createContext("light") // default value
```

`createContext` returns a context object with a `Provider` component. The argument is the default value used when no provider is above the consumer.

## Providing Context Values

Wrap the part of your tree that needs the value with the Provider:

```jsx
function App() {
  const [theme, setTheme] = useState("dark")

  return (
    <ThemeContext.Provider value={theme}>
      <Layout />
    </ThemeContext.Provider>
  )
}
```

Every component inside the Provider can access the `theme` value.

## Consuming Context with useContext

```jsx
function Button() {
  const theme = useContext(ThemeContext)
  return <button className={`btn-${theme}`}>Click</button>
}
```

No props needed. `Button` reads `theme` directly from the context. `Layout` and `Sidebar` do not even know about `theme` anymore.

## A Complete Example: Theme Toggle

```jsx
const ThemeContext = createContext()

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light")

  const toggle = () => {
    setTheme(prev => prev === "light" ? "dark" : "light")
  }

  return (
    <ThemeContext.Provider value={{ theme, toggle }}>
      {children}
    </ThemeContext.Provider>
  )
}

function ThemedButton() {
  const { theme, toggle } = useContext(ThemeContext)
  return (
    <button onClick={toggle} className={`btn-${theme}`}>
      Toggle Theme
    </button>
  )
}

function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
    </ThemeProvider>
  )
}
```

Wrapping the Provider in its own component keeps things clean and reusable.

## When to Use Context

Context is great for data that many components need:

- Theme (light/dark mode)
- Auth state (logged in user info)
- Language/locale settings
- App-wide configuration

## When NOT to Use Context

Context is not a replacement for all state management. Every time the context value changes, every component that consumes it re-renders. If you put too much data in one context, you cause unnecessary re-renders.

For complex state that changes frequently, consider splitting into multiple smaller contexts or using a state management library like Zustand.

## Tips

- Create a custom hook for consuming context to add error checking:

```jsx
function useTheme() {
  const context = useContext(ThemeContext)
  if (context === undefined) {
    throw new Error("useTheme must be used within a ThemeProvider")
  }
  return context
}
```

- Put the Provider as high as needed, but no higher. Only wrap the subtree that needs the data.

- Co-locate the context definition, provider, and custom hook in the same file for easy imports.
