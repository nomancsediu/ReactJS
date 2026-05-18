# Rules of Hooks

Hooks have two strict rules. Break them and your app will crash or behave unpredictably in ways that are very hard to debug. I learned these the hard way. Let me make sure you do not have to.

## Rule 1: Only Call Hooks at the Top Level

Do not call hooks inside loops, conditions, or nested functions. Always call them at the top level of your component function, before any early returns.

```jsx
// WRONG - hook inside a condition
function BadComponent({ isLoggedIn }) {
  if (isLoggedIn) {
    const [user, setUser] = useState(null) // breaks the rules!
  }
  return <div>Hello</div>
}

// CORRECT - hook at top level, condition inside hook logic
function GoodComponent({ isLoggedIn }) {
  const [user, setUser] = useState(null)

  useEffect(() => {
    if (isLoggedIn) {
      fetchUser().then(setUser)
    }
  }, [isLoggedIn])

  return <div>{user ? user.name : "Guest"}</div>
}
```

Why does this matter? React relies on the order of hook calls. Each render, React needs to match each hook call with its previous state. If hooks are called in a different order between renders, the matching breaks and state gets mixed up.

Consider what happens with a conditional hook:

```jsx
// First render: hooks called in order [useState, useState, useEffect]
// Second render (condition false): hooks called in order [useState, useEffect]
// React thinks the second useState from render 1 is the useEffect from render 2
// State corruption!
```

## Rule 2: Only Call Hooks from React Functions

Call hooks from:

- Function components
- Custom hooks

Do not call hooks from:

- Regular JavaScript functions
- Class components
- Event handlers (call them in the component body, not inside a click handler)

```jsx
// WRONG - hook in a regular function
function getUser() {
  const [user, setUser] = useState(null) // not a React function!
  return user
}

// CORRECT - hook in a custom hook (starts with "use")
function useUser() {
  const [user, setUser] = useState(null)
  useEffect(() => {
    fetchUser().then(setUser)
  }, [])
  return user
}
```

## The ESLint Plugin

The `eslint-plugin-react-hooks` plugin catches rule violations automatically. It has two rules:

- `rules-of-hooks` - enforces the two rules above
- `exhaustive-deps` - warns about missing effect dependencies

Set it up in your project:

```json
{
  "plugins": ["react-hooks"],
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

If you use Vite with the React template, this is already configured. Keep it on. The warnings it produces are almost always real problems.

## Common Violations

**1. Conditional useEffect:**

```jsx
// Wrong
if (userId) {
  useEffect(() => { fetchUser(userId) }, [userId])
}

// Right
useEffect(() => {
  if (userId) fetchUser(userId)
}, [userId])
```

**2. Hooks in callbacks:**

```jsx
// Wrong
const handleClick = () => {
  const [value, setValue] = useState(0) // no!
}

// Right - call the hook at top level, use it in the callback
const [value, setValue] = useState(0)
const handleClick = () => {
  setValue(prev => prev + 1)
}
```

**3. Hooks in loops:**

```jsx
// Wrong
items.forEach(item => {
  const [expanded, setExpanded] = useState(false) // no!
})

// Right - move state into a child component
function Item({ item }) {
  const [expanded, setExpanded] = useState(false)
  return <div onClick={() => setExpanded(!expanded)}>{item.name}</div>
}
```

Follow the rules, use the ESLint plugin, and hooks will work reliably. Ignore them, and you will spend hours chasing bugs that make no sense. These rules exist for a reason. Respect them.
