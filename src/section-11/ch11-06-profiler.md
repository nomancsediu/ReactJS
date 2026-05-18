# React Profiler

You cannot fix what you cannot measure. The React Profiler is your best friend for finding performance problems. It shows you exactly which components render, how long they take, and why they rendered. I used to guess at performance fixes. Now I measure first.

## Using the DevTools Profiler

Install the React Developer Tools extension for your browser. Open DevTools and find the Profiler tab.

1. Click the record button (circle icon)
2. Interact with your app (click buttons, type, navigate)
3. Click the record button again to stop

You will see a flame chart showing every component that rendered and how long each took.

## Reading the Profiler

The flame chart uses color to show render time:

- **Gray**: Did not render
- **Green**: Rendered quickly
- **Yellow**: Rendered slowly
- **Red**: Very slow, needs attention

Click on any component to see details:

- **Why did this render?** Shows the cause (props changed, state changed, hook changed)
- **Render duration** How long the render took
- **When did this render?** Which interaction triggered it

## Finding Bottlenecks

Here is a real example. I had a form that felt sluggish when typing:

```jsx
function Form() {
  const [text, setText] = useState("");

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <ExpensivePreview text={text} />       {/* Slow! */}
      <StaticSidebar items={bigList} />       {/* Re-renders for no reason */}
      <Footer />                              {/* Also re-renders */}
    </div>
  );
}
```

The Profiler showed that `ExpensivePreview`, `StaticSidebar`, and `Footer` all re-rendered on every keystroke. The fix was simple:

```jsx
const ExpensivePreview = React.memo(function Preview({ text }) {
  // Only re-renders when text changes
  return <div>{heavyTransform(text)}</div>;
});

const StaticSidebar = React.memo(function Sidebar({ items }) {
  return <nav>{items.map(...)}</nav>;
});

const Footer = React.memo(function Footer() {
  return <footer>...</footer>;
});
```

Typing became instant because only the input and preview re-rendered.

## Programmatic Profiling

You can also profile in code using the `Profiler` component:

```jsx
import { Profiler } from "react";

function onRender(id, phase, actualDuration) {
  console.log(`${id} ${phase} took ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="Dashboard" onRender={onRender}>
      <Dashboard />
    </Profiler>
  );
}
```

This is useful for tracking performance in production. You can send the data to your analytics service.

## Profiling Tips I Use

- Record specific interactions, not random browsing
- Compare before and after your optimizations
- Focus on components that render most often or take the longest
- Check the "Why did this render?" panel before optimizing
- Profile in production mode for accurate numbers (development is slower)

The Profiler turns performance debugging from guessing into measuring. Use it every time you think something might be slow. You will often find the problem is not where you expected.
