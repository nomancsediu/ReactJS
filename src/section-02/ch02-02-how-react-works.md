# How React Works

Understanding how React works under the hood helped me write better code. You do not need to know every detail, but a mental model of the process prevents a lot of confusion. Let me explain the key ideas.

## The Virtual DOM

React keeps a copy of the actual DOM in memory. This is the Virtual DOM. It is just a JavaScript object that represents your UI.

When your component renders, React creates a Virtual DOM tree. When state or props change, React creates a new Virtual DOM tree and compares it with the previous one. This comparison is called reconciliation.

```jsx
// First render
<div>
  <h1>Hello</h1>
  <p>Count: 0</p>
</div>

// After state change (count becomes 1)
<div>
  <h1>Hello</h1>
  <p>Count: 1</p>  // Only this changed
</div>
```

React sees that only the `<p>` text changed, so it updates just that one DOM node. The `<h1>` is left alone. This is why React is fast for most use cases.

## Reconciliation

The reconciliation algorithm determines what changed between two Virtual DOM trees. It follows two rules:

1. **Different element types produce different trees.** If a `<div>` becomes a `<span>`, React tears down the whole subtree and rebuilds it.
2. **The developer hints at stable identity with the `key` prop.** This is why keys matter in lists. Without keys, React cannot tell which items moved, got added, or got removed.

```jsx
// Without keys, React cannot match items efficiently
items.map(item => <li>{item.name}</li>)

// With keys, React knows which item is which
items.map(item => <li key={item.id}>{item.name}</li>)
```

## Fiber

Fiber is React's rendering engine, introduced in React 16. It enables a key feature: work can be split into chunks and paused.

Before Fiber, rendering was synchronous. Once React started updating the DOM, it could not stop. With large UIs, this caused dropped frames and janky scrolling.

Fiber introduced a priority system. High-priority work like user input gets handled first. Low-priority work like data fetching updates can wait. React can pause, resume, and reorder work as needed.

You rarely interact with Fiber directly. Just know that it is the reason React stays responsive even in complex apps.

## The Rendering Cycle

Here is what happens when you update state:

1. You call `setState` or dispatch an action
2. React schedules a re-render
3. React calls your component function to get new Virtual DOM
4. It compares the new Virtual DOM with the previous one
5. It calculates the minimal set of DOM changes needed
6. It applies those changes to the real DOM

Steps 3 through 5 are the "render phase." Step 6 is the "commit phase." Only during the commit phase does the actual DOM get touched.

## Why This Matters

Knowing this helps you understand:

- Why you should not mutate state directly (React would not detect the change)
- Why keys matter (reconciliation uses them to match elements)
- Why rendering can happen multiple times before commit (React can abort renders)
- Why expensive computations in render slow things down (they run every render)

This mental model will guide you as we dive into components, state, and effects.
