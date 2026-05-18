# Props

Props are how you pass data from a parent component to a child component. Think of them like function arguments. The parent provides the data, the child receives it and uses it to render.

## Passing Props

You pass props as attributes on the component tag, just like HTML attributes:

```jsx
function App() {
  return <UserCard name="Alex" age={25} />
}

function UserCard(props) {
  return (
    <div>
      <p>Name: {props.name}</p>
      <p>Age: {props.age}</p>
    </div>
  )
}
```

The `props` object contains all the attributes you passed. `props.name` is "Alex", `props.age` is 25.

Notice that `age={25}` uses curly braces. That is because it is a number, not a string. Without curly braces, it would be the string `"25"`.

## Destructuring Props

Most React code destructures props in the parameter for cleaner code:

```jsx
function UserCard({ name, age }) {
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
    </div>
  )
}
```

Same result, less typing, easier to read.

## Default Props

You can set default values right in the destructuring:

```jsx
function Alert({ message, type = "info" }) {
  return <div className={`alert alert-${type}`}>{message}</div>
}

// type defaults to "info" when not provided
<Alert message="File saved" />
// type is "error" when provided
<Alert message="Something broke" type="error" />
```

## The children Prop

`children` is a special prop that contains whatever you put between the opening and closing tags of a component:

```jsx
function Card({ children }) {
  return <div className="card">{children}</div>
}

// Usage
<Card>
  <h2>Title</h2>
  <p>Some content here</p>
</Card>
```

The `children` prop makes components flexible. The `Card` does not need to know what is inside it. It just wraps whatever you give it.

## Spread Props

If you have an object with many properties, you can spread them as props instead of writing each one:

```jsx
const user = { name: "Alex", age: 25, role: "admin" }

// Instead of this
<UserCard name={user.name} age={user.age} role={user.role} />

// You can do this
<UserCard {...user} />
```

Both produce the same result. Use spread when it makes code cleaner, not as a lazy shortcut to avoid thinking about what data you are passing.

## Props Are Read-Only

A component must never modify its props. Props flow one direction: from parent to child. If a child needs to tell the parent something, it does so through callback functions passed as props:

```jsx
function Parent() {
  const handleClick = (id) => console.log("Clicked:", id)
  return <Child onClick={handleClick} />
}

function Child({ onClick }) {
  return <button onClick={() => onClick(42)}>Click me</button>
}
```

Props are the communication channel between components. Keep them flowing down, use callbacks to flow up, and your app stays predictable and easy to debug.
