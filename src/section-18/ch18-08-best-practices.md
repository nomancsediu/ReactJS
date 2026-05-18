# Best Practices Checklist

After writing this entire book, I wanted to put together a checklist of everything I have learned. These are the practices I try to follow in every React project. I do not always get them right, but having a checklist helps me stay honest.

## Component Design

- Keep components small and focused on one thing
- Extract reusable logic into custom hooks
- Use composition over props for flexibility
- Prefer named exports over default exports
- Keep prop interfaces simple and well-typed

```tsx
// Good: small, focused, typed
interface UserCardProps {
  name: string;
  email: string;
  avatar?: string;
}

function UserCard({ name, email, avatar }: UserCardProps) {
  return (
    <div className="p-4 border rounded">
      {avatar && <img src={avatar} alt={name} className="w-12 h-12 rounded-full" />}
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
}
```

## State Management

- Lift state up only as high as needed, not higher
- Use local state by default, context when sharing across components
- Keep derived state out of state (compute it during render)
- Use `useReducer` for complex state logic
- Avoid unnecessary state that can be calculated

```tsx
// Bad: storing derived state
const [items, setItems] = useState([]);
const [itemCount, setItemCount] = useState(0);

// Good: compute derived values
const [items, setItems] = useState([]);
const itemCount = items.length;
```

## Performance

- Use `memo` only when profiling shows a real problem
- Use `useMemo` and `useCallback` sparingly, not by default
- Lazy load routes and heavy components
- Virtualize long lists (react-window or react-virtuoso)
- Optimize images with `next/image` or lazy loading

```tsx
// Lazy load a heavy component
const HeavyChart = lazy(() => import("./HeavyChart"));

function Dashboard() {
  return (
    <Suspense fallback={<p>Loading chart...</p>}>
      <HeavyChart />
    </Suspense>
  );
}
```

## Error Handling

- Wrap risky components in error boundaries
- Use try/catch for async operations
- Show meaningful error messages to users
- Log errors to a monitoring service
- Handle loading, error, and empty states in every data component

```tsx
function DataComponent() {
  const { data, loading, error } = useFetch("/api/data");

  if (loading) return <Skeleton />;
  if (error) return <ErrorMessage message={error} />;
  if (!data || data.length === 0) return <EmptyState />;

  return <DataList items={data} />;
}
```

## Security

- Never trust user input, sanitize before rendering
- Use `dangerouslySetInnerHTML` only with sanitized content
- Keep secrets on the server, never in client bundles
- Validate data on both client and server
- Set appropriate CORS and CSP headers

## Code Quality

- Use TypeScript for type safety
- Run a linter (ESLint) on every commit
- Use Prettier for consistent formatting
- Write meaningful component and function names
- Add comments only for "why," not "what"

```tsx
// Bad: comment explains what the code already says
// Increment count by 1
setCount(count + 1);

// Good: comment explains why
// Round trip to server to validate before closing
await validateBeforeClose();
```

## Developer Experience

- Set up CI/CD from the start
- Use absolute imports with path aliases
- Keep dependencies up to date
- Document setup steps in a README
- Use environment variables for configuration

## Accessibility

- Use semantic HTML elements
- Add alt text to all images
- Ensure keyboard navigation works
- Test with a screen reader occasionally
- Use sufficient color contrast

## Testing

- Write integration tests for important user flows
- Use Testing Library, not Enzyme
- Test behavior, not implementation
- Keep tests independent of each other
- Run tests in CI

This checklist is not exhaustive, and I do not follow every item perfectly on every project. But having it written down keeps me accountable. When I review my code, I can scan this list and ask, "Did I handle errors here? Is this accessible? Should this be memoized?" That habit has improved my code more than any single technique.
