# Arrow Functions

Arrow functions took me a while to get used to. They looked weird. Now I use them constantly, especially in React. Let me walk you through how they work.

## Basic Syntax

A regular function looks like this:

```js
function add(a, b) {
  return a + b
}
```

The arrow function equivalent:

```js
const add = (a, b) => {
  return a + b
}
```

Shorter, right? The `=>` is the arrow. That is where the name comes from.

## Implicit Return

If your function body is a single expression, you can skip the curly braces and the `return` keyword. The expression is returned automatically.

```js
const add = (a, b) => a + b
const double = (n) => n * 2
const greet = (name) => `Hello, ${name}!`
```

This is called implicit return. I use this all the time for simple operations.

One parameter? You can even drop the parentheses:

```js
const double = n => n * 2
```

No parameters? You need empty parentheses:

```js
const sayHi = () => console.log("Hi!")
```

## The `this` Binding

This is the big one. Regular functions create their own `this` context. Arrow functions do not. They inherit `this` from the surrounding code.

```js
const team = {
  name: "Alpha",
  members: ["Alex", "Sam"],

  showMembers() {
    this.members.forEach(function(member) {
      // "this" is wrong here! It is not the team object
      console.log(`${member} is in ${this.name}`) // this.name is undefined
    })
  },

  showMembersFixed() {
    this.members.forEach((member) => {
      // Arrow function inherits "this" from showMembersFixed
      console.log(`${member} is in ${this.name}`) // this.name is "Alpha"
    })
  }
}
```

Before arrow functions, people used workarounds like `var self = this`. Arrow functions solve this cleanly.

## When to Use Arrow Functions

- Use them for callbacks and short helper functions
- Use them when you want to inherit `this` from the parent scope
- Avoid them for object methods where you need `this` to refer to the object

In React, arrow functions are everywhere:

```jsx
const Button = () => {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>Clicked {count} times</button>
}
```

Get comfortable with arrow functions. You will type them more than any other syntax in React.
