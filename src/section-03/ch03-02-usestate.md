# useState in Depth

You already saw `useState` in the fundamentals section. Now let me go deeper into how it really works, edge cases, and patterns you will use in real projects.

## Quick Recap

```jsx
const [value, setValue] = useState(initialValue)
```

- `value` is the current state
- `setValue` is the function to update it
- `initialValue` is what state starts as on the first render

## Lazy Initialization

If your initial state requires an expensive calculation, pass a function instead of a value. React only calls this function on the first render:

```jsx
// Bad - runs on every render
const [data, setData] = useState(expensiveCalculation())

// Good - runs only once on the first render
const [data, setData] = useState(() => expensiveCalculation())
```

This matters when the initial value comes from `localStorage`, parsing JSON, or any computation:

```jsx
const [theme, setTheme] = useState(() => {
  return localStorage.getItem("theme") || "light"
})
```

Without the function wrapper, `localStorage.getItem` runs on every render, even though the result is ignored after the first one.

## State Updates with Objects

State updates replace the value. They do not merge. This is a common source of bugs:

```jsx
const [user, setUser] = useState({ name: "Alex", age: 25, city: "Portland" })

// Wrong - loses age and city
setUser({ name: "Sam" }) // state becomes { name: "Sam" }, age and city gone!

// Correct - spread first, then override
setUser({ ...user, name: "Sam" }) // { name: "Sam", age: 25, city: "Portland" }
```

Always spread the existing state before overriding specific properties.

## Functional Updates for Objects

When updating objects based on previous state, use the functional form:

```jsx
setUser(prev => ({ ...prev, age: prev.age + 1 }))
```

This guarantees you are working with the latest state, even if multiple updates are queued.

## Arrays in State

Same principle. Do not mutate. Create a new array:

```jsx
const [items, setItems] = useState([])

// Adding an item
setItems([...items, newItem])           // append
setItems([newItem, ...items])           // prepend

// Removing an item
setItems(items.filter(item => item.id !== idToRemove))

// Updating an item
setItems(items.map(item =>
  item.id === updatedId ? { ...item, ...updates } : item
))
```

## Multiple State Variables

You can use `useState` multiple times in one component:

```jsx
function Form() {
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [rememberMe, setRememberMe] = useState(false)

  // ...
}
```

This is often cleaner than one big state object because each value updates independently.

## State Is Local

State belongs to the component that declares it. If the component unmounts, the state is gone. If the component renders twice, each instance has its own state:

```jsx
function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>
}

function App() {
  return (
    <div>
      <Counter /> {/* independent count */}
      <Counter /> {/* separate independent count */}
    </div>
  )
}
```

Each `Counter` has its own `count`. They do not share state.

`useState` is the hook you will use most. Master its patterns and the rest of React becomes much easier.
