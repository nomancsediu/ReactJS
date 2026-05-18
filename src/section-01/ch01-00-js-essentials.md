# Why JS Essentials Matter Before React

When I first tried learning React, I jumped straight into components and JSX. Bad idea. I kept getting confused by things that were not React problems at all. They were JavaScript problems.

React is not a new language. It is JavaScript with a particular way of organizing your code. If your JavaScript foundation is shaky, React will feel impossible. But if you understand the JS concepts underneath, React starts to make real sense.

Here is what I learned the hard way. You need to be comfortable with these before React will click:

- **Variables and scoping** so you know where your data lives
- **Arrow functions** because React code is full of them
- **Destructuring** since props and state use it everywhere
- **Array methods** especially `map` and `filter` for rendering lists
- **Promises and async/await** for fetching data
- **ES modules** for organizing your code across files

Think of it this way. React is the house. JavaScript is the ground it sits on. You would not build a house on sand, right? Same idea here.

```js
// This looks like React magic if you don't know destructuring
function UserCard({ name, email }) {
  return <p>{name} - {email}</p>
}

// But once you know JS destructuring, it is just this:
const { name, email } = props
```

I spent weeks fighting React because I kept running into "why does this not work" moments. Almost every time, the answer was a JavaScript concept I had skipped.

This section covers the JavaScript I wish I had learned first. Each chapter focuses on one concept with examples that connect directly to React. You do not need to be a JS expert. You just need enough to not get stuck on the basics while learning React.

If some of these concepts are already familiar, skim through and move on. But if anything feels new, take your time. Your future self will thank you when React code starts reading like plain English.
