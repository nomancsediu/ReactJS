# Array Methods

Array methods are the bread and butter of React. You will use `map` on every single project, probably every day. I want to cover the ones you will see most often, with examples that feel like real React code.

## map - Transform Each Element

`map` creates a new array by running a function on every element. This is how you render lists in React.

```js
const numbers = [1, 2, 3, 4]
const doubled = numbers.map(n => n * 2) // [2, 4, 6, 8]
```

In React:

```jsx
const names = ["Alex", "Sam", "Jordan"]
return (
  <ul>
    {names.map(name => <li key={name}>{name}</li>)}
  </ul>
)
```

## filter - Keep Only Matching Elements

`filter` returns a new array with only the elements that pass a test:

```js
const scores = [45, 80, 92, 30, 75]
const passing = scores.filter(score => score >= 60) // [80, 92, 75]
```

In React:

```jsx
const [users, setUsers] = useState([
  { name: "Alex", active: true },
  { name: "Sam", active: false },
  { name: "Jordan", active: true }
])

const activeUsers = users.filter(user => user.active)
```

## reduce - Accumulate Into One Value

`reduce` folds the array into a single value. It takes a callback with an accumulator and the current element:

```js
const cart = [
  { item: "Shirt", price: 30 },
  { item: "Pants", price: 50 },
  { item: "Hat", price: 20 }
]

const total = cart.reduce((sum, product) => sum + product.price, 0) // 100
```

## find - Get the First Match

Returns the first element that satisfies the condition, or `undefined` if nothing matches:

```js
const users = [
  { id: 1, name: "Alex" },
  { id: 2, name: "Sam" }
]

const user = users.find(u => u.id === 2) // { id: 2, name: "Sam" }
```

## some and every - Boolean Checks

`some` returns `true` if at least one element passes. `every` returns `true` if all elements pass:

```js
const ages = [18, 22, 16, 30]
ages.some(age => age < 18)  // true (one minor)
ages.every(age => age >= 16) // true (all at least 16)
```

## forEach - Just Loop

`forEach` runs a function on each element but does not return anything. Use it for side effects only, not for creating new arrays:

```js
const names = ["Alex", "Sam"]
names.forEach(name => console.log(name))
```

Avoid `forEach` in JSX. Use `map` for rendering because it returns an array React can display.

## The Key Takeaway

`map` and `filter` are your daily drivers in React. `reduce` and `find` come up often too. Practice these until they are second nature. Every list you render and every filtered view you build will use them.
