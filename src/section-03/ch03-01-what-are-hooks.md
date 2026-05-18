# What Are Hooks?

Hooks are functions that let you "hook into" React features from function components. Before hooks, only class components could have state and lifecycle methods. Hooks fixed that limitation and changed how we write React.

## The Problem Before Hooks

Class components worked, but they had issues:

```jsx
// A simple counter as a class component
class Counter extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
    this.increment = this.increment.bind(this)
  }

  increment() {
    this.setState({ count: this.state.count + 1 })
  }

  render() {
    return <button onClick={this.increment}>{this.state.count}</button>
  }
}
```

That is a lot of boilerplate for a counter. The `constructor`, `super`, `bind`, and `this` everywhere add noise. Now compare with hooks:

```jsx
function Counter() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

Much cleaner. Same functionality, less code, no `this` confusion.

## What Hooks Solve

**1. No more `this` binding.** Function components do not have `this`. Period.

**2. Related logic stays together.** In class components, related code was split across lifecycle methods. `componentDidMount` and `componentDidUpdate` might contain pieces of the same logic. Hooks let you keep related code in one place.

**3. Sharing logic is easier.** Before hooks, sharing stateful logic required patterns like render props and higher-order components. They worked but were awkward. Custom hooks solve this cleanly.

**4. Smaller bundle sizes.** Class components have overhead. Function components with hooks compile to smaller code.

## Built-in Hooks

React provides several hooks out of the box:

| Hook | Purpose |
|------|---------|
| `useState` | Add state to a component |
| `useEffect` | Run side effects |
| `useRef` | Access DOM or persist values |
| `useMemo` | Memoize expensive computations |
| `useCallback` | Memoize functions |
| `useReducer` | Manage complex state |
| `useContext` | Access context values |

Plus you can create your own custom hooks by combining these.

## Hook Rules Overview

There are two rules you must follow:

1. **Only call hooks at the top level.** Do not call them inside loops, conditions, or nested functions. Always call them at the top of your component function.

2. **Only call hooks from React functions.** Call them from function components or custom hooks, not regular JavaScript functions.

```jsx
// Wrong - hook inside a condition
if (isLoggedIn) {
  const [user, setUser] = useState(null)
}

// Correct - hook at the top level, condition inside the hook
const [user, setUser] = useState(null)
```

We will cover the rules in detail later. For now, just know they exist and matter.

Hooks are not a fancy feature you can ignore. They are the standard way to write React. Every component you build will use them. Understanding hooks is understanding modern React.
