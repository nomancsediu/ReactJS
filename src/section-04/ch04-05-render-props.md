# Render Props

The render props pattern was one of those things that looked weird the first time I saw it. Passing a function as a prop that returns JSX? But once I understood the problem it solves, it became one of my favorite tools.

## What Is a Render Prop?

A render prop is a function prop that a component calls to determine what to render. Instead of the component knowing what to show, it delegates that decision to the caller.

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return render(position);
}

// Usage
<MouseTracker
  render={({ x, y }) => (
    <h1>Mouse is at {x}, {y}</h1>
  )}
/>
```

The `MouseTracker` handles the mouse tracking logic. The `render` prop decides what to show with that data. You get the logic for free and control the presentation.

## Function as Children

A popular variation uses `children` as the render prop:

```jsx
function MouseTracker({ children }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);

  return children(position);
}

// Usage
<MouseTracker>
  {({ x, y }) => <div>Cursor: {x}, {y}</div>}
</MouseTracker>
```

This reads a bit cleaner. The function is passed as the child, and the component calls it with the data.

## Sharing Logic Between Components

Render props shine when you need the same logic in different UIs:

```jsx
function DataFetcher({ url, children }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(json => {
        setData(json);
        setLoading(false);
      });
  }, [url]);

  return children({ data, loading });
}

// Same logic, different UI
<DataFetcher url="/api/users">
  {({ data, loading }) =>
    loading ? <Spinner /> : <UserList users={data} />
  }
</DataFetcher>

<DataFetcher url="/api/posts">
  {({ data, loading }) =>
    loading ? <Skeleton /> : <PostGrid posts={data} />
  }
</DataFetcher>
```

## When to Use Render Props

I reach for render props when:

- Multiple components need the same stateful logic but different UI
- I want to share behavior without forcing a specific rendering
- A component needs to expose internal state to its parent

Render props give you maximum flexibility. The component provides the logic, and the caller provides the view.
