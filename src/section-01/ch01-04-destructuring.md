# Destructuring

Destructuring lets you pull values out of objects and arrays in a clean, readable way. When I first saw it, the syntax looked confusing. Now I cannot imagine writing JS without it. React uses destructuring everywhere, especially for props.

## Object Destructuring

```js
const user = { name: "Alex", age: 25, city: "Portland" }

// Without destructuring
const name = user.name
const age = user.age

// With destructuring
const { name, age } = user
```

You can also rename variables while destructuring:

```js
const { name: userName, age: userAge } = user
// Now you use userName and userAge instead
```

## Default Values

If a property might be missing, you can provide a fallback:

```js
const user = { name: "Alex" }
const { name, role = "viewer" } = user
// role is "viewer" because it was not in the object
```

## Nested Destructuring

You can dig into nested objects too:

```js
const response = {
  data: {
    user: {
      email: "alex@test.com"
    }
  }
}

const { data: { user: { email } } } = response
// email is "alex@test.com"
```

Be careful with deep nesting though. If any level is undefined, the whole thing throws an error. Optional chaining (covered later) helps with that.

## Array Destructuring

Arrays use position instead of names:

```js
const colors = ["red", "green", "blue"]
const [first, second, third] = colors

// Skip elements
const [primary, , tertiary] = colors // skips "green"
```

A common pattern is swapping variables without a temp:

```js
let a = 1, b = 2;
[a, b] = [b, a] // a is 2, b is 1
```

## Destructuring in Function Parameters

This is where it shines in React:

```js
// Without destructuring
function UserCard(props) {
  return `${props.name} - ${props.email}`
}

// With destructuring in the parameter
function UserCard({ name, email }) {
  return `${name} - ${email}`
}
```

You can combine defaults with parameter destructuring:

```js
function UserCard({ name, role = "member" }) {
  return `${name} is a ${role}`
}
```

## In React Hooks

Array destructuring is how hooks work:

```jsx
const [count, setCount] = useState(0)
// useState returns an array, and we destructure it into two named variables
```

Destructuring is not optional knowledge for React. It is woven into almost every pattern you will write. Practice it until it feels natural.
