# Events

User interactions in React are handled through events. Clicks, typing, form submissions, all of it. React events look like regular DOM events but work a bit differently. Let me show you the patterns you will use every day.

## Basic Event Handling

You attach event handlers as props on elements. The prop name uses camelCase:

```jsx
function Button() {
  const handleClick = () => {
    console.log("Button was clicked")
  }

  return <button onClick={handleClick}>Click me</button>
}
```

Notice: you pass the function reference, not the result of calling it. `onClick={handleClick}` is correct. `onClick={handleClick()}` calls the function immediately, which is wrong.

For inline handlers, use an arrow function:

```jsx
<button onClick={() => setCount(count + 1)}>Add</button>
```

## Common Events

**onChange** for text inputs:

```jsx
function SearchBox() {
  const [query, setQuery] = useState("")

  return (
    <input
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      placeholder="Search..."
    />
  )
}
```

**onSubmit** for forms:

```jsx
function LoginForm() {
  const [email, setEmail] = useState("")

  const handleSubmit = (e) => {
    e.preventDefault() // prevent page reload
    console.log("Submitted:", email)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button type="submit">Log in</button>
    </form>
  )
}
```

Always call `e.preventDefault()` in form handlers to stop the browser from reloading the page.

**Other useful events:**

- `onMouseEnter` / `onMouseLeave` for hover effects
- `onFocus` / `onBlur` for input focus
- `onKeyDown` / `onKeyUp` for keyboard input
- `onDoubleClick` for double-click actions

## The Event Object

React passes a synthetic event object to your handler. It has the same interface as a native DOM event but works consistently across browsers:

```jsx
function InputExample() {
  const handleKeyDown = (e) => {
    console.log("Key pressed:", e.key)
    console.log("Ctrl held:", e.ctrlKey)

    if (e.key === "Enter") {
      console.log("Enter was pressed")
    }
  }

  return <input onKeyDown={handleKeyDown} />
}
```

Common properties: `e.target`, `e.key`, `e.type`, `e.preventDefault()`, `e.stopPropagation()`.

## Passing Arguments to Handlers

Sometimes you need to pass extra data to an event handler:

```jsx
function TodoList({ todos, onDelete }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          {todo.text}
          <button onClick={() => onDelete(todo.id)}>Delete</button>
        </li>
      ))}
    </ul>
  )
}
```

The arrow function wrapper lets you pass `todo.id` while still receiving the event properly.

## Handler Patterns

For complex logic, define the handler function separately instead of inline:

```jsx
function Form() {
  const [value, setValue] = useState("")

  const handleChange = (e) => {
    const input = e.target.value
    if (input.length <= 100) {
      setValue(input)
    }
  }

  return <textarea value={value} onChange={handleChange} />
}
```

This keeps your JSX clean and lets you add validation or side effects easily.

Events in React are straightforward once you know the patterns. Use camelCase names, pass function references, and call `preventDefault` when you need to stop default browser behavior.
