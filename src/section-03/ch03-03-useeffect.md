# useEffect

`useEffect` lets you run side effects in function components. Fetching data, setting up subscriptions, manipulating the DOM, timers, these all go in `useEffect`. It is powerful but also the hook that causes the most confusion. Let me break it down.

## Basic Usage

```jsx
useEffect(() => {
  // This runs after every render
  document.title = `Count: ${count}`
})
```

The function you pass to `useEffect` is called the "effect." It runs after the component renders.

## The Dependency Array

The second argument controls when the effect runs:

```jsx
// No dependency array - runs after every render
useEffect(() => {
  console.log("Runs every render")
})

// Empty array - runs only once after the first render
useEffect(() => {
  console.log("Runs once on mount")
}, [])

// With dependencies - runs when dependencies change
useEffect(() => {
  console.log(`User ID changed to ${userId}`)
}, [userId])
```

The dependency array is how you tell React "only re-run this effect when these values change."

## Cleanup Function

Some effects need cleanup. Subscriptions, timers, event listeners, they all need to be stopped when the component unmounts or before the effect runs again. Return a cleanup function for this:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setSeconds(prev => prev + 1)
  }, 1000)

  // Cleanup runs when component unmounts or before effect re-runs
  return () => clearInterval(id)
}, [])
```

Without the cleanup, the interval keeps running even after the component disappears from the screen. This causes memory leaks and bugs.

## Fetching Data

The most common use case:

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    let cancelled = false

    async function fetchUser() {
      setLoading(true)
      const response = await fetch(`/api/users/${userId}`)
      const data = await response.json()

      if (!cancelled) {
        setUser(data)
        setLoading(false)
      }
    }

    fetchUser()

    return () => {
      cancelled = true // prevent setting state on unmounted component
    }
  }, [userId])

  if (loading) return <p>Loading...</p>
  return <h1>{user.name}</h1>
}
```

The `cancelled` flag prevents updating state if the component unmounted or `userId` changed before the fetch completed.

## Common Pitfalls

**1. Missing dependencies.** If you use a variable inside the effect but leave it out of the dependency array, you get stale closures. The ESLint `react-hooks/exhaustive-deps` rule catches this.

**2. Infinite loops.** Setting state inside an effect with no dependency array causes a render loop: render, effect runs, setState, re-render, effect runs, and so on.

**3. Running effects too often.** If your dependency is an object or array created in the component body, it is a new reference every render, and the effect runs every time. Memoize it or move it outside the component.

**4. Not cleaning up.** Forgetting cleanup for subscriptions and timers is a classic memory leak.

## Mental Model

Think of `useEffect` as telling React: "after rendering, do this thing." The dependency array adds "but only if these values changed." The cleanup adds "and stop doing the previous thing before starting the new one."

`useEffect` is essential but tricky. When something goes wrong with it, check your dependency array first. Nine times out of ten, that is where the problem lives.
