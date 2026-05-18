# useRef

`useRef` is the hook I understood last, but it fills an important gap. It gives you a mutable value that persists across renders without causing re-renders when it changes. Think of it as a box that holds a value and stays the same box between renders.

## Basic Syntax

```jsx
const myRef = useRef(initialValue)
```

`myRef.current` holds the value. You can read and write `myRef.current` directly. Changing it does not trigger a re-render.

## Accessing DOM Elements

The most common use case is getting a direct reference to a DOM element:

```jsx
function TextInput() {
  const inputRef = useRef(null)

  const focusInput = () => {
    inputRef.current.focus()
  }

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus the input</button>
    </div>
  )
}
```

You attach the ref to an element with the `ref` attribute. After the component renders, `inputRef.current` points to the actual DOM node.

## Persisting Values Without Re-renders

Sometimes you need a value that survives re-renders but should not trigger one when updated. `useRef` is perfect for this:

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0)
  const intervalRef = useRef(null)

  const start = () => {
    if (intervalRef.current !== null) return // already running
    intervalRef.current = setInterval(() => {
      setSeconds(prev => prev + 1)
    }, 1000)
  }

  const stop = () => {
    clearInterval(intervalRef.current)
    intervalRef.current = null
  }

  return (
    <div>
      <p>{seconds} seconds</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  )
}
```

If we used `useState` to store the interval ID, setting it would cause an unnecessary re-render. With `useRef`, changing the interval ID does not affect rendering.

## useRef vs useState

| Feature | useState | useRef |
|---------|----------|--------|
| Triggers re-render on change | Yes | No |
| Persists across renders | Yes | Yes |
| Mutable | No (use setter) | Yes (`.current`) |
| Use for | Data the UI displays | Data the UI does not display |

Simple rule: if the value affects what appears on screen, use `useState`. If it does not, use `useRef`.

## Tracking Previous Values

A clever `useRef` pattern is storing the previous value of a prop or state:

```jsx
function usePrevious(value) {
  const ref = useRef()

  useEffect(() => {
    ref.current = value
  })

  return ref.current
}

// Usage
function PriceDisplay({ price }) {
  const prevPrice = usePrevious(price)
  const trend = price > prevPrice ? "up" : price < prevPrice ? "down" : "same"

  return (
    <div>
      <p>Current: ${price}</p>
      <p>Trend: {trend}</p>
    </div>
  )
}
```

The `useEffect` runs after render, so `ref.current` still holds the previous value during the current render.

## Avoid These Mistakes

- Do not read or set `ref.current` during rendering. It should only be used in effects and event handlers.
- Do not use `useRef` as a way to avoid `useState` just because re-renders are inconvenient. If the UI needs to reflect the value, it must be state.

`useRef` is simple but essential. It bridges the gap between React's declarative world and the imperative DOM API.
