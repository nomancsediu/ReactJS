# useMemo

`useMemo` caches the result of a calculation so it does not get recomputed on every render. It is a performance optimization tool. The key question is: do you actually need it? Let me explain when `useMemo` helps and when it does not.

## Basic Syntax

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b])
```

- The first argument is a function that returns the value you want to memoize
- The second argument is the dependency array
- React only recomputes the value when a dependency changes

## A Practical Example

```jsx
function ProductFilter({ products, searchQuery }) {
  const filteredProducts = useMemo(() => {
    console.log("Filtering products...")
    return products.filter(product =>
      product.name.toLowerCase().includes(searchQuery.toLowerCase())
    )
  }, [products, searchQuery])

  return (
    <ul>
      {filteredProducts.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  )
}
```

Without `useMemo`, the filter runs on every render, even if `products` and `searchQuery` have not changed. With `useMemo`, it only runs when one of the dependencies changes.

## When to Use useMemo

**1. Expensive calculations.** If the computation takes noticeable time and runs frequently, memoize it.

```jsx
const sortedData = useMemo(() => {
  return [...largeArray].sort((a, b) => a.name.localeCompare(b.name))
}, [largeArray])
```

**2. Referential equality for objects passed as props.** Every render creates new object and array references. If you pass these to a memoized child component, the memoization breaks:

```jsx
function Parent() {
  const config = useMemo(() => ({ theme: "dark", size: "md" }), [])
  return <Child config={config} />
}
```

Without `useMemo`, `config` is a new object every render, and `React.memo(Child)` would re-render unnecessarily.

**3. Values used as dependencies in other hooks.** If you create an object and use it in a `useEffect` dependency array, the effect runs every render because the object reference changes. `useMemo` stabilizes the reference.

## When NOT to Use useMemo

Most computations are fast. Memoization has its own cost: React has to check dependencies and manage the cache. For simple operations, memoization slows things down:

```jsx
// Do not memoize this - string concatenation is instant
const greeting = useMemo(() => `Hello, ${name}`, [name])

// Just compute it directly
const greeting = `Hello, ${name}`
```

A good rule: write the code without `useMemo` first. If you notice a performance problem, then add memoization where it helps.

## Dependency Array Gotchas

Dependencies are compared with `Object.is`. This means objects and arrays are compared by reference, not by content:

```jsx
// This creates a new array every render, so useMemo recomputes every time
const items = useMemo(() => compute(data), [data])
// If data is a new array each render, memoization is useless
```

Make sure your dependencies are stable. Use `useMemo` on the dependencies themselves if needed.

## The Bottom Line

`useMemo` is for optimization, not for correctness. Your app should work the same with or without it. Use it when profiling shows a real performance issue. Do not sprinkle it everywhere "just in case."
