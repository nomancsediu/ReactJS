# React.memo and Memoization

When a parent re-renders, all its children re-render too. Most of the time, that is fine. But when a child is expensive or the tree is deep, those unnecessary renders add up. `React.memo` lets you tell React "only re-render this component if its props actually changed."

## How React.memo Works

`React.memo` wraps your component and does a shallow comparison of props. If nothing changed, it skips the render:

```jsx
const ExpensiveList = React.memo(function ItemList({ items, onSelect }) {
  console.log("ItemList rendered");
  return items.map((item) => (
    <div key={item.id} onClick={() => onSelect(item.id)}>
      {item.name}
    </div>
  ));
});
```

Now this component only re-renders when `items` or `onSelect` actually change.

## The Shallow Comparison Trap

`React.memo` uses shallow comparison. That means it compares props with `===`. This causes problems with objects and functions:

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // This creates a NEW array every render
  const items = [{ id: 1, name: "Apple" }, { id: 2, name: "Banana" }];

  // This creates a NEW function every render
  const handleSelect = (id) => console.log(id);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <ExpensiveList items={items} onSelect={handleSelect} />
    </div>
  );
}
```

Even with `React.memo`, this re-renders every time because `items` and `handleSelect` are new references on each render. Shallow comparison sees them as different.

## Fixing It with useMemo and useCallback

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // Memoize the array so it stays the same reference
  const items = useMemo(
    () => [{ id: 1, name: "Apple" }, { id: 2, name: "Banana" }],
    [] // dependencies
  );

  // Memoize the function so it stays the same reference
  const handleSelect = useCallback((id) => {
    console.log(id);
  }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
      <ExpensiveList items={items} onSelect={handleSelect} />
    </div>
  );
}
```

Now `ExpensiveList` truly skips re-rendering when only `count` changes.

## When to Use React.memo

Use it when:

- The component renders often with the same props
- The component is expensive to render
- The component is deep in the tree and receives stable props

## The Danger of Over-Memoizing

I went through a phase where I memoized everything. Every component got `React.memo`, every value got `useMemo`, every function got `useCallback`. My code was harder to read and the app was not faster.

Memoization has a cost. React has to compare props on every render. If the comparison cost is higher than the render cost, you made things slower.

```jsx
// Bad: memoizing a trivial component
const SimpleLabel = React.memo(({ text }) => <span>{text}</span>);
// The comparison costs more than just rendering the span

// Good: memoizing an expensive component
const DataTable = React.memo(({ rows, columns, onSort }) => {
  // Heavy rendering with many rows
  return <table>...</table>;
});
```

The rule is simple: measure first, memo second. If you cannot prove a component is slow, do not memoize it. The React Profiler will tell you what actually needs help.
