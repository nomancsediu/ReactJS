# useCallback

`useCallback` memoizes a function so it keeps the same reference across renders. It is closely related to `useMemo`. In fact, `useCallback(fn, deps)` is equivalent to `useMemo(() => fn, deps)`. But it has specific use cases that are worth understanding.

## Basic Syntax

```jsx
const memoizedFunction = useCallback(() => {
  doSomething(a, b)
}, [a, b])
```

The function is recreated only when a dependency changes. Otherwise, the same function reference is reused.

## Why Function References Matter

Every time a component renders, any function defined inside it gets a new reference:

```jsx
function Parent() {
  const [count, setCount] = useState(0)

  // This is a new function every render
  const handleClick = () => {
    console.log("clicked")
  }

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child onClick={handleClick} />
    </div>
  )
}
```

Even though `handleClick` does the same thing every time, it is a brand new function object on each render. If `Child` is wrapped in `React.memo`, it will re-render anyway because `onClick` changed.

## When to Use useCallback

**1. Passing callbacks to memoized child components.**

```jsx
const MemoizedChild = React.memo(Child)

function Parent() {
  const [count, setCount] = useState(0)

  const handleClick = useCallback(() => {
    console.log("clicked")
  }, []) // same function reference every render

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <MemoizedChild onClick={handleClick} />
    </div>
  )
}
```

Now `MemoizedChild` only re-renders when its props actually change.

**2. Functions used as dependencies in useEffect.**

```jsx
function SearchResults({ query }) {
  const fetchData = useCallback(async () => {
    const response = await fetch(`/api/search?q=${query}`)
    return response.json()
  }, [query])

  useEffect(() => {
    fetchData()
  }, [fetchData]) // stable reference prevents infinite loops
}
```

**3. Functions passed to custom hooks that depend on referential equality.**

## Common Mistakes

**Mistake 1: Wrapping every function in useCallback.**

```jsx
// Unnecessary - this component has no memoized children
function SimpleButton() {
  const handleClick = useCallback(() => {
    console.log("clicked")
  }, [])
  return <button onClick={handleClick}>Click</button>
}
```

If the child is not memoized, `useCallback` provides zero benefit. The child re-renders regardless.

**Mistake 2: Stale closures from missing dependencies.**

```jsx
const [count, setCount] = useState(0)

// Bug - count is always 0 inside this callback
const logCount = useCallback(() => {
  console.log(count)
}, []) // missing count dependency

// Fix - add count to dependencies
const logCount = useCallback(() => {
  console.log(count)
}, [count])
```

**Mistake 3: Over-optimizing.** If you are wrapping everything in `useCallback` without measuring, you are adding complexity for no gain. Profile first, optimize second.

## useCallback vs useMemo

- `useCallback` caches a function
- `useMemo` caches the result of calling a function

```jsx
const fn = useCallback(() => compute(a, b), [a, b])  // caches the function
const value = useMemo(() => compute(a, b), [a, b])    // caches the result
```

Use `useCallback` when you need the same function reference. Use `useMemo` when you need the same computed value.
