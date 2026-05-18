# JSX Deep Dive

JSX is the syntax that makes React feel unique. It looks like HTML inside JavaScript, but it is neither HTML nor a string template. Understanding what JSX actually is will save you from many confusing moments.

## What Is JSX?

JSX is a syntax extension for JavaScript. It lets you write UI structure in a way that looks like HTML:

```jsx
const element = <h1>Hello, world!</h1>
```

The browser does not understand JSX. A build tool (like Vite with Babel) converts it into regular JavaScript:

```jsx
// What you write
const element = <h1 className="title">Hello</h1>

// What it becomes
const element = React.createElement("h1", { className: "title" }, "Hello")
```

`React.createElement` returns an object that React uses to create DOM elements. JSX is just a nicer way to write those `createElement` calls.

## Expressions in JSX

Use curly braces `{}` to embed any JavaScript expression:

```jsx
const name = "Alex"
const element = <h1>Hello, {name}!</h1>

// Any expression works
const total = <p>Price: ${(9.99 * 1.08).toFixed(2)}</p>
const greeting = <span>{isLoggedIn ? "Welcome back" : "Please sign in"}</span>
```

Curly braces are for expressions only, not statements. You cannot put `if` or `for` inside them directly. Use ternary operators and `map` instead.

## JSX Rules

JSX has some differences from HTML that trip up beginners:

**1. className instead of class**

```jsx
// Wrong
<div class="card">

// Correct
<div className="card">
```

`class` is a reserved word in JavaScript, so JSX uses `className`.

**2. Self-closing tags need a slash**

```jsx
// Wrong
<img src="photo.jpg">
<br>

// Correct
<img src="photo.jpg" />
<br />
```

**3. Inline styles use camelCase objects**

```jsx
// Wrong
<div style="color: red; font-size: 16px">

// Correct
<div style={{ color: "red", fontSize: "16px" }}>
```

**4. JSX must return a single parent element**

```jsx
// Wrong - two siblings with no parent
return (
  <h1>Title</h1>
  <p>Text</p>
)

// Correct - wrapped in a fragment
return (
  <>
    <h1>Title</h1>
    <p>Text</p>
  </>
)
```

Fragments (`<>...</>`) let you group elements without adding extra DOM nodes.

**5. All tags must be closed**

```jsx
// Wrong
<input type="text">

// Correct
<input type="text" />
```

## Conditional JSX

You cannot use `if/else` inside curly braces, but you have options:

```jsx
// Ternary for either/or
return isLoggedIn ? <Dashboard /> : <LoginPage />

// && for show/hide
return (
  <div>
    <h1>Welcome</h1>
    {isAdmin && <AdminPanel />}
  </div>
)
```

JSX feels strange at first, but it quickly becomes natural. Think of it as a way to describe your UI structure that happens to look like HTML.
