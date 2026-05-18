# State

State is how a component remembers things. Props come from the parent and cannot change. State belongs to the component itself and can update over time. When state changes, the component re-renders to show the new data.

## useState

The `useState` hook gives a component its own state. It returns an array with two items: the current value and a function to update it.

```jsx
import { useState } from "react"

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Add 1</button>
    </div>
  )
}
```

`count` starts at `0` (the initial value). `setCount` updates it. When `setCount` is called, the component re-renders with the new value.

## State Updates Are Asynchronous

When you call `setCount`, the value does not change immediately. React schedules an update and re-renders later:

```jsx
function Example() {
  const [count, setCount] = useState(0)

  const handleClick = () => {
    setCount(count + 1)
    console.log(count) // Still the old value!
  }
}
```

This catches a lot of beginners off guard. The `console.log` runs before the state actually updates.

## Using the Previous State

When your new state depends on the old state, use the functional form of the setter:

```jsx
// Risky - might use a stale value
setCount(count + 1)

// Safe - always uses the latest value
setCount(prev => prev + 1)
```

This matters especially if you update state multiple times in the same event handler:

```jsx
const handleClick = () => {
  setCount(count + 1) // might be 0 + 1 = 1
  setCount(count + 1) // might still be 0 + 1 = 1
}

// With functional updates
const handleClick = () => {
  setCount(prev => prev + 1) // 0 + 1 = 1
  setCount(prev => prev + 1) // 1 + 1 = 2
}
```

## State Batching

React batches state updates for performance. Multiple `setState` calls in the same event handler trigger only one re-render:

```jsx
const handleClick = () => {
  setFirstName("Alex")
  setLastName("Smith")
  setAge(25)
  // Only one re-render happens, not three
}
```

This is good. It keeps your app fast.

## Objects in State

When state is an object, you must replace the whole object, not mutate it:

```jsx
const [user, setUser] = useState({ name: "Alex", age: 25 })

// Wrong - mutating state directly
user.name = "Sam"
setUser(user) // React might not detect the change

// Correct - create a new object
setUser({ ...user, name: "Sam" }) // spread + override
```

Always create a new object or array when updating state. This is why the spread operator matters so much in React.

## What Should Be State?

Not everything needs to be state. Ask yourself: does this value change over time, and does the UI need to update when it changes? If yes, it is state. If no, use a regular variable or a ref instead.
