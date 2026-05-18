# Conditional Rendering

React does not have special template syntax for showing or hiding things. You just use regular JavaScript. This felt odd to me at first, but it is actually simpler and more flexible than template engines.

## The Ternary Operator

Use ternary when you want to show one thing or another:

```jsx
function Greeting({ isLoggedIn }) {
  return isLoggedIn ? <Dashboard /> : <LoginPage />
}
```

This is the most common pattern. Condition on the left, two possible outcomes on the right.

## The && Operator

Use `&&` when you want to show something or nothing:

```jsx
function Mailbox({ unreadMessages }) {
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 && (
        <p>You have {unreadMessages.length} unread messages.</p>
      )}
    </div>
  )
}
```

If the condition is `false`, React renders nothing. If `true`, it renders the JSX after `&&`.

Be careful with numbers. `0 && <Component />` renders `0` because `0` is a falsy value that React will display. Use a boolean or comparison instead:

```jsx
// Problem
{items.length && <ItemList items={items} />}  // might render "0"

// Fix
{items.length > 0 && <ItemList items={items} />}
```

## if/else Outside JSX

For more complex logic, use a regular `if` statement before the return:

```jsx
function UserPanel({ user, loading, error }) {
  if (loading) return <Spinner />
  if (error) return <ErrorMessage error={error} />
  if (!user) return <LoginPage />

  return (
    <div>
      <h2>Welcome, {user.name}</h2>
      <UserStats stats={user.stats} />
    </div>
  )
}
```

Early returns keep the main rendering logic clean and unindented.

## Storing JSX in Variables

You can also assign JSX to variables for complex conditions:

```jsx
function Notification({ type, message }) {
  let icon

  if (type === "success") {
    icon = <CheckCircleIcon />
  } else if (type === "warning") {
    icon = <AlertIcon />
  } else {
    icon = <InfoIcon />
  }

  return (
    <div className={`notification ${type}`}>
      {icon}
      <span>{message}</span>
    </div>
  )
}
```

## show/hide with CSS

Sometimes you want to keep a component mounted but hidden:

```jsx
function TabContent({ activeTab, tab }) {
  return (
    <div style={{ display: activeTab === tab ? "block" : "none" }}>
      {/* Component stays mounted, just hidden */}
    </div>
  )
}
```

This preserves state inside the hidden component. Use this when switching tabs and you do not want to lose user input.

## Which Approach to Use?

- **Ternary** for simple either/or choices
- **&&** for show/hide without an alternative
- **if/else** for multiple conditions or early returns
- **CSS hiding** when you want to keep components mounted

There is no one right way. Pick whatever makes your code easiest to read for that specific situation.
