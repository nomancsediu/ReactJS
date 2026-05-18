# Optional Chaining and Nullish Coalescing

These two features saved me from writing so many `if` checks. They handle the common problem of accessing data that might be `undefined` or `null`. In React, where you often deal with API responses that have unpredictable shapes, these operators are lifesavers.

## Optional Chaining: `?.`

The `?.` operator stops and returns `undefined` if it hits a `null` or `undefined` value, instead of throwing an error.

```js
const user = { name: "Alex" }

// Without optional chaining
const street = user.address && user.address.street // undefined, no error

// With optional chaining
const street = user.address?.street // undefined, no error
```

Before `?.`, accessing `user.address.street` when `address` was undefined would crash your app. Now it safely returns `undefined`.

Works with function calls too:

```js
const obj = { greet: () => "hello" }

obj.greet?.()  // "hello"
obj.farewell?.() // undefined, no crash
```

And with arrays:

```js
const items = null
items?.[0] // undefined instead of error
```

## Nullish Coalescing: `??`

The `??` operator returns the right side only when the left side is `null` or `undefined`. Not for other falsy values like `0` or `""`.

```js
const count = 0
const display = count ?? 10  // 0 (because 0 is not null/undefined)

const name = null
const displayName = name ?? "Guest" // "Guest"
```

Compare this with `||` which treats any falsy value as missing:

```js
const count = 0
const display = count || 10  // 10 (because 0 is falsy)
// This is probably a bug! 0 might be a valid value.
```

Use `??` when you want a fallback only for `null` and `undefined`. Use `||` when any falsy value should trigger the fallback.

## Combining Both

These operators work great together:

```js
const user = { profile: { theme: null } }

const theme = user.profile?.theme ?? "light" // "light"
```

## In React

API data often arrives incomplete. Optional chaining prevents crashes:

```jsx
function UserCard({ user }) {
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.company?.name ?? "No company"}</p>
      <p>{user.address?.city ?? "Unknown location"}</p>
    </div>
  )
}
```

Without `?.` and `??`, you would need nested checks like:

```jsx
<p>{user.company && user.company.name ? user.company.name : "No company"}</p>
```

Much harder to read. Optional chaining and nullish coalescing make this code clean and safe. Use them whenever you access data that might not exist.
