# Template Literals

String concatenation used to be annoying. You had to break out of the string, add a plus sign, add the variable, and start a new string. Template literals fix all of that. They are one of my favorite JS features.

## Basic Interpolation

Use backticks (`` ` ``) instead of quotes. Put variables inside `${}`:

```js
const name = "Alex"
const age = 25

// Old way
const msg1 = "My name is " + name + " and I am " + age + " years old."

// Template literal
const msg2 = `My name is ${name} and I am ${age} years old.`
```

So much cleaner. The `${}` part evaluates any JavaScript expression, not just variables:

```js
const price = 19.99
const tax = 0.08
const total = `Total: $${(price * (1 + tax)).toFixed(2)}` // "Total: $21.59"
```

## Multiline Strings

Regular strings cannot span multiple lines without `\n`. Template literals can:

```js
const html = `
  <div class="card">
    <h2>Hello</h2>
    <p>Welcome back</p>
  </div>
`
```

All the line breaks and spacing are preserved. This is super useful when building HTML strings or formatting text.

## Tagged Templates

This is an advanced feature I rarely use, but it is good to know it exists. A tag function lets you process a template literal before it becomes a string.

```js
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const val = values[i] ? `<mark>${values[i]}</mark>` : ""
    return result + str + val
  }, "")
}

const keyword = "React"
const output = highlight`Learning ${keyword} is fun`
// "Learning <mark>React</mark> is fun"
```

The tag function receives the string parts and the interpolated values separately. Libraries like styled-components use this technique heavily.

## Practical React Usage

You will see template literals constantly in React for className combinations and dynamic text:

```jsx
const Button = ({ variant, children }) => {
  const className = `btn btn-${variant}`
  return <button className={className}>{children}</button>
}
```

Template literals are simple but powerful. Once you start using backticks, you will never want to go back to plus signs for string concatenation.
