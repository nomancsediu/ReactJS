# let, const, and var

When I started JavaScript, I used `var` for everything. It worked, so why care? Then I ran into bugs that made no sense, and it turned out `var` was the culprit. Let me explain the differences so you can avoid my mistakes.

## var: The Old Way

`var` is function-scoped. That means it is accessible anywhere inside the function where it is declared, even before the declaration line.

```js
console.log(greeting) // undefined (hoisted)
var greeting = "hello"

function example() {
  if (true) {
    var x = 10
  }
  console.log(x) // 10 - var leaks out of the if block
}
```

This "leaking" out of blocks causes real bugs. You might reuse a variable name and accidentally overwrite something.

## let: The Block-Scoped Variable

`let` is block-scoped. It stays inside the curly braces where you define it.

```js
function example() {
  if (true) {
    let x = 10
  }
  console.log(x) // ReferenceError! x does not exist here
}
```

Use `let` when you need to reassign a value later.

```js
let count = 0
count = count + 1 // perfectly fine
```

## const: The Constant

`const` is also block-scoped, but you cannot reassign it after the initial value.

```js
const name = "Alex"
name = "Sam" // TypeError! Assignment to constant variable
```

Important detail: `const` prevents reassignment, not mutation. Objects and arrays inside a `const` can still be modified.

```js
const colors = ["red", "blue"]
colors.push("green") // this works!
colors = ["yellow"]   // TypeError! cannot reassign

const user = { name: "Alex" }
user.name = "Sam"  // this works!
```

## When to Use Each

Here is my simple rule:

- Use `const` by default. It tells other developers "this value will not change."
- Use `let` when you genuinely need to reassign, like counters or swapping values.
- Avoid `var` in modern code. It exists for backward compatibility.

In React, you will see `const` everywhere because most values do not get reassigned:

```js
const [count, setCount] = useState(0) // const, not let
const handleClick = () => { /* ... */ } // const for functions
```

Pick `const` first. Switch to `let` only when you must.
