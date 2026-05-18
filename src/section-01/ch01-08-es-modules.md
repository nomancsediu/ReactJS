# ES Modules

When your project grows, you need to split code into separate files. ES modules are the standard way to do that in JavaScript. React projects rely on modules heavily, so understanding import and export is essential.

## Named Exports

You can export multiple things from one file using named exports:

```js
// math.js
export const add = (a, b) => a + b
export const subtract = (a, b) => a - b
export const multiply = (a, b) => a * b
```

Import them by name using curly braces:

```js
// app.js
import { add, multiply } from "./math.js"

add(2, 3)       // 5
multiply(4, 5)  // 20
```

You can import and rename at the same time:

```js
import { add as sum } from "./math.js"
sum(2, 3) // 5
```

## Default Export

Each file can have one default export. No curly braces needed when importing:

```js
// Button.js
const Button = ({ label }) => <button>{label}</button>
export default Button
```

```js
// App.js
import Button from "./Button.js"
```

You can name the default import whatever you want:

```js
import MyButton from "./Button.js" // works fine
```

## Combining Both

A file can have one default export and multiple named exports:

```js
// utils.js
export const formatName = (name) => name.toUpperCase()
export const formatDate = (date) => new Date(date).toLocaleDateString()
const helpers = { formatName, formatDate }
export default helpers
```

```js
import helpers, { formatName } from "./utils.js"
```

## Re-exports

You can re-export from another file, which is useful for creating index files:

```js
// components/index.js
export { default as Button } from "./Button.js"
export { default as Card } from "./Card.js"
export { default as Input } from "./Input.js"
```

```js
import { Button, Card, Input } from "./components"
```

This pattern is common in React projects. It keeps imports tidy.

## Dynamic Imports

You can load modules on demand using dynamic `import()`. This returns a promise:

```js
async function loadChart() {
  const { Chart } = await import("./heavy-chart-module.js")
  // Chart is loaded only when this function runs
}
```

Dynamic imports enable code splitting, which helps performance by only loading code when needed.

## In React

Every React component file follows this pattern:

```jsx
// UserProfile.jsx
import { useState } from "react"          // from a package
import { fetchUser } from "../api"         // from your code
import Avatar from "./Avatar"              // default import

export default function UserProfile({ userId }) {
  const [user, setUser] = useState(null)
  // ...
}
```

Modules are not optional in React. They are how every file communicates. Learn the syntax well and your project structure will stay clean.
