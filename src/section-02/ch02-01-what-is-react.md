# What Is React?

React is a JavaScript library for building user interfaces. Created by Facebook in 2013, it changed how developers think about building web apps. I want to explain what makes React different and why it became so popular.

## The Problem React Solves

Before React, updating the UI meant manually finding DOM elements and changing them. Something like this:

```js
document.getElementById("counter").textContent = newCount
document.getElementById("message").className = count > 10 ? "highlight" : ""
```

This gets messy fast. When your app grows, you have UI state scattered everywhere, and keeping the DOM in sync with your data becomes a nightmare. React takes a different approach.

## Declarative UI

With React, you describe what the UI should look like for a given state. When state changes, React figures out how to update the DOM for you.

```jsx
function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  )
}
```

You never touch the DOM directly. You just say "here is what the UI looks like when count is 5" and React makes it happen. This is called declarative programming.

## The Component Model

React apps are built from components. A component is a reusable piece of UI. Think of them like LEGO blocks:

```jsx
function App() {
  return (
    <div>
      <Header />
      <Sidebar />
      <MainContent />
      <Footer />
    </div>
  )
}
```

Each component manages its own logic and appearance. You compose small components into bigger ones. This keeps your code organized and your UI consistent.

## Why Was React Created?

Facebook had a problem. Their apps had complex, interactive UIs with data that changed frequently. Ads appeared, notifications updated, chat messages arrived, all while the user was scrolling. Managing all those DOM updates was error-prone and slow.

React's solution was simple: re-render everything when data changes, but be smart about what actually changes in the DOM. That "being smart" part is the Virtual DOM, which we will cover next.

## The React Ecosystem

React is small. It handles rendering UI and managing state. For everything else, the community built solutions:

- **React Router** for navigation
- **Redux or Zustand** for complex state
- **Next.js** for server rendering
- **TanStack Query** for data fetching

You do not need all of these. Start with plain React and add tools when you actually need them.

React is popular because it is simple at its core. Components, state, and re-rendering. That is the foundation everything else builds on.
