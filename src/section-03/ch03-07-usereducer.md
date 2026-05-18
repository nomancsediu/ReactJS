# useReducer

`useReducer` is an alternative to `useState` for managing complex state logic. If you find yourself with multiple related state values or state updates that depend on the previous state in complicated ways, `useReducer` might be the better choice.

## Basic Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState)
```

- `reducer` is a function that takes the current state and an action, then returns the new state
- `initialState` is the starting state value
- `dispatch` is the function you call to send actions to the reducer

## A Simple Example

```jsx
const initialState = { count: 0 }

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 }
    case "decrement":
      return { count: state.count - 1 }
    case "reset":
      return { count: 0 }
    default:
      return state
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState)

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "reset" })}>Reset</button>
    </div>
  )
}
```

Instead of calling `setCount`, you dispatch an action describing what happened. The reducer decides how the state should change.

## Actions with Payloads

Actions can carry extra data:

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "add_item":
      return { items: [...state.items, action.payload] }
    case "remove_item":
      return { items: state.items.filter(item => item.id !== action.payload) }
    default:
      return state
  }
}

// Dispatching with a payload
dispatch({ type: "add_item", payload: { id: 1, name: "Milk" } })
dispatch({ type: "remove_item", payload: 1 })
```

## A More Realistic Example: Form State

Forms often have multiple related fields. `useReducer` keeps the logic organized:

```jsx
const initialForm = {
  email: "",
  password: "",
  rememberMe: false,
  errors: {}
}

function formReducer(state, action) {
  switch (action.type) {
    case "update_field":
      return {
        ...state,
        [action.field]: action.value,
        errors: { ...state.errors, [action.field]: undefined }
      }
    case "set_errors":
      return { ...state, errors: action.errors }
    case "reset":
      return initialForm
    default:
      return state
  }
}

function LoginForm() {
  const [form, dispatch] = useReducer(formReducer, initialForm)

  const handleChange = (field, value) => {
    dispatch({ type: "update_field", field, value })
  }

  return (
    <form>
      <input
        value={form.email}
        onChange={(e) => handleChange("email", e.target.value)}
      />
      <input
        type="password"
        value={form.password}
        onChange={(e) => handleChange("password", e.target.value)}
      />
    </form>
  )
}
```

## useReducer vs useState

Choose `useState` when:

- State is simple (a single value or a few unrelated values)
- Updates are straightforward

Choose `useReducer` when:

- State involves multiple related values
- The next state depends heavily on the previous state
- State transitions follow clear patterns (like a form or a game)
- You want to separate state logic from the component

`useReducer` takes more setup but scales better for complex state. Think of it as useState with a traffic cop directing all the updates through a single, organized function.
