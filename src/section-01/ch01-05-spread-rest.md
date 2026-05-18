# Spread and Rest Operators

The spread and rest operators both use three dots (`...`). They look the same but do opposite things. Spread expands something out. Rest collects things together. Let me show you both.

## Spread with Arrays

Spread "spreads out" the elements of an array:

```js
const a = [1, 2, 3]
const b = [4, 5, 6]
const combined = [...a, ...b] // [1, 2, 3, 4, 5, 6]
```

You can also add elements while spreading:

```js
const numbers = [1, 2, 3]
const withZero = [0, ...numbers] // [0, 1, 2, 3]
const withEnd = [...numbers, 4]  // [1, 2, 3, 4]
```

This is how you copy arrays without mutating the original:

```js
const original = [1, 2, 3]
const copy = [...original] // a new array, not a reference
```

## Spread with Objects

Same idea for objects. It copies properties from one object into another:

```js
const defaults = { theme: "light", lang: "en" }
const userPrefs = { lang: "fr", fontSize: 16 }
const settings = { ...defaults, ...userPrefs }
// { theme: "light", lang: "fr", fontSize: 16 }
```

Later properties overwrite earlier ones. This is why `lang` became "fr".

## Rest Parameters

Rest collects multiple elements into a single array. Use it in function parameters:

```js
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0)
}
sum(1, 2, 3, 4) // 10
```

Rest must be the last parameter:

```js
function greet(greeting, ...names) {
  return `${greeting}, ${names.join(" and ")}!`
}
greet("Hello", "Alex", "Sam") // "Hello, Alex and Sam!"
```

## Rest in Destructuring

You can also use rest while destructuring to collect the remaining items:

```js
const [first, ...rest] = [1, 2, 3, 4, 5]
// first is 1, rest is [2, 3, 4, 5]

const { name, ...otherInfo } = { name: "Alex", age: 25, city: "Portland" }
// name is "Alex", otherInfo is { age: 25, city: "Portland" }
```

## In React

Spread is everywhere in React for updating state immutably:

```jsx
const [user, setUser] = useState({ name: "Alex", age: 25 })

const haveBirthday = () => {
  setUser({ ...user, age: user.age + 1 }) // copy all, then overwrite age
}
```

And for passing props:

```jsx
const props = { name: "Alex", role: "admin" }
<UserCard {...props} />
// Same as <UserCard name="Alex" role="admin" />
```

Remember: spread copies shallowly. Nested objects are still references. For deep copies, you need a different approach.
