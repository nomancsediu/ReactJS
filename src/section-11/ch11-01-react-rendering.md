# When React Renders

Understanding when React renders is the key to making your app fast. I used to think React was magic that just worked. Then I wondered why my app was slow, and realized I had no idea when renders happened. Let me break it down.

## What Triggers a Render

A component renders when one of three things happens:

1. **Its state changes** (via `setState` or the setter from `useState`)
2. **Its props change** (a parent re-rendered and passed new props)
3. **Its parent re-renders** (even if props did not change)

That third one surprised me. A parent re-rendering causes all children to re-render, even if nothing changed for them.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <Child name="Alice" />  {/* Re-renders every time count changes! */}
    </div>
  );
}

function Child({ name }) {
  console.log("Child rendered"); // This fires on every count change
  return <p>Hello {name}</p>;
}
```

Even though `name` never changes, `Child` re-renders because `Parent` re-renders.

## The Render Cycle

Here is what happens when state changes:

1. **Schedule** - React marks the component as needing an update
2. **Render** - React calls your component function to get new virtual DOM
3. **Commit** - React compares the new virtual DOM with the old one and updates the real DOM

The render step is the expensive part. It runs your component code. The commit step is usually fast because React only changes what actually differs.

## Unnecessary Renders

Not all renders are equal. A render that produces the same output is wasted work:

```jsx
function ExpensiveChild({ items }) {
  // This component does heavy computation
  const processed = items.map(item => ({
    ...item,
    score: heavyCalculation(item)
  }));

  return processed.map(item => <ItemRow key={item.id} item={item} />);
}

function Parent() {
  const [search, setSearch] = useState("");
  // Typing in search re-renders ExpensiveChild
  // even though items did not change!
  return (
    <div>
      <input value={search} onChange={(e) => setSearch(e.target.value)} />
      <ExpensiveChild items={bigItemsList} />
    </div>
  );
}
```

Every keystroke re-renders `ExpensiveChild` and runs that heavy computation again. That is the kind of slowness users notice.

## How to Spot Unnecessary Renders

Add a console log to see what renders when:

```jsx
function MyComponent(props) {
  console.log("MyComponent rendered", props);
  return <div>...</div>;
}
```

Or use the React DevTools Profiler to see which components render and how long they take. We will cover that in detail later.

The goal is not zero renders. React needs to render to update the screen. The goal is to eliminate renders that do nothing useful. Once you see them, you can fix them with `React.memo`, `useMemo`, and `useCallback`, which we cover next.
