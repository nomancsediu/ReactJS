# Closures and Scope

Closures sounded scary when I first read about them. The word itself feels academic. But the concept is actually straightforward, and understanding it will save you from some really confusing React bugs.

## Lexical Scope

JavaScript uses lexical scoping. This means a function can access variables from the place where it was defined, not where it is called.

```js
function outer() {
  const message = "hello"

  function inner() {
    console.log(message) // "hello" - inner remembers message
  }

  return inner
}

const fn = outer()
fn() // "hello" - even though outer() has finished running
```

`inner` "closes over" the `message` variable. That is a closure. A function that remembers the variables from its birthplace, even after that place has finished executing.

## Why Closures Matter

They let you create private data. Variables trapped in a closure cannot be accessed from outside:

```js
function createCounter() {
  let count = 0 // private to the closure

  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count
  }
}

const counter = createCounter()
counter.increment() // 1
counter.increment() // 2
counter.getCount()  // 2
// count is not accessible directly
```

## Common Pitfall: Stale Closures

This is where React developers get tripped up. A closure captures the value at the time it was created, not the current value:

```js
function setup() {
  let count = 0

  setTimeout(() => {
    console.log(count) // 0, not 2
  }, 1000)

  count = 2
}
```

The timeout callback captured `count` as `0`. Changing `count` later does not update the closure.

## Closures in React Hooks

This happens all the time with React state:

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0)

  useEffect(() => {
    const id = setInterval(() => {
      // This closure captures "seconds" as 0
      // It will always log 0, never 1, 2, 3...
      console.log(seconds)
    }, 1000)
    return () => clearInterval(id)
  }, []) // empty dependency = closure captures initial value
}
```

The fix is to use the functional form of `setState`:

```jsx
setSeconds(prev => prev + 1) // prev is always the latest value
```

Or add `seconds` to the dependency array so the effect re-runs with the current value.

## The Mental Model

Think of a closure as a snapshot. When a function is created, it takes a snapshot of all the variables it can see. That snapshot does not automatically update. In React, this is why the dependency array in `useEffect` matters. It tells React when to take a new snapshot.

Closures are not something to fear. They are just functions remembering where they came from. Once you see them clearly, a whole category of React bugs stops being mysterious.
