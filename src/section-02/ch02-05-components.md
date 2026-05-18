# Components

Components are the building blocks of every React app. Once you understand components, everything else in React starts making sense. Let me walk you through how they work.

## What Is a Component?

A component is a JavaScript function that returns JSX. That is it. It takes inputs (called props) and describes what should appear on screen.

```jsx
function Greeting() {
  return <h1>Hello there!</h1>
}
```

This is a function component. It is the standard way to write components in modern React.

## Component Anatomy

A typical component has three parts: the function declaration, optional logic, and the return statement with JSX:

```jsx
function UserCard({ name, role }) {
  const displayRole = role || "member"

  return (
    <div className="card">
      <h2>{name}</h2>
      <p>Role: {displayRole}</p>
    </div>
  )
}
```

You can add any JavaScript logic before the return. Variables, conditionals, helper function calls, anything you need.

## Using Components

You use a component by writing it like an HTML tag with a capital letter:

```jsx
function App() {
  return (
    <div>
      <UserCard name="Alex" role="admin" />
      <UserCard name="Sam" role="editor" />
      <UserCard name="Jordan" />
    </div>
  )
}
```

Each `<UserCard />` is an independent instance with its own data.

## Naming Conventions

- Component names must start with a capital letter. `UserCard` is a component. `userCard` is not.
- Use PascalCase: `NavBar`, `SearchInput`, `ProductList`
- Name files to match: `UserCard.jsx`, `SearchInput.jsx`

React uses capitalization to tell the difference between native HTML tags (`div`, `span`) and your custom components (`UserCard`, `NavBar`).

## Exporting Components

Each component usually lives in its own file and is exported:

```jsx
// Button.jsx
export default function Button({ label, onClick }) {
  return <button onClick={onClick}>{label}</button>
}
```

Then imported where needed:

```jsx
// App.jsx
import Button from "./Button"

function App() {
  return <Button label="Save" onClick={() => console.log("saved")} />
}
```

## Components Can Nest

Components can contain other components. This is composition:

```jsx
function App() {
  return (
    <Layout>
      <Header />
      <Main>
        <Sidebar />
        <Content />
      </Main>
      <Footer />
    </Layout>
  )
}
```

Think of it like Russian dolls. Small components go inside bigger ones, which go inside even bigger ones. This keeps each piece focused and manageable.

## A Simple Rule

If a piece of UI is used in multiple places, or if a component is getting long and hard to read, extract it into its own component. There is no strict rule for when to split. Use your judgment. When a component does too many things, it is time to break it up.

Components are simple. They are functions that return UI. The power comes from composing them together to build complex interfaces from small, understandable pieces.
