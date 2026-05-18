# Error Boundaries

Sometimes things go wrong. A network request fails, a component gets unexpected data, or you just have a typo. Without error handling, one broken component can crash your entire app. Error boundaries are how React deals with this.

## The Problem

Normally, a JavaScript error inside a component unmounts the entire React tree. The user sees a blank white page. Not great.

```tsx
// This crashes the whole app if user is null
function Profile({ user }: { user: User }) {
  return <h1>{user.name}</h1>; // TypeError: Cannot read property 'name'
}
```

## Error Boundary to the Rescue

An error boundary is a class component that catches errors in its children. When an error occurs, it shows a fallback UI instead of crashing:

```tsx
import { Component, ReactNode } from "react";

interface ErrorBoundaryProps {
  fallback?: ReactNode;
  children: ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log the error to an error reporting service
    console.error("Error caught by boundary:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div className="p-4 bg-red-50 rounded">
            <h2>Something went wrong</h2>
            <p>{this.state.error?.message}</p>
            <button onClick={() => this.setState({ hasError: false, error: null })}>
              Try again
            </button>
          </div>
        )
      );
    }

    return this.props.children;
  }
}
```

## Using an Error Boundary

Wrap any component that might fail:

```tsx
function App() {
  return (
    <div>
      <Header />
      <ErrorBoundary fallback={<p>Failed to load profile.</p>}>
        <Profile user={undefined as any} />
      </ErrorBoundary>
      <Footer />
    </div>
  );
}
```

Now if `Profile` crashes, only that section shows the fallback. `Header` and `Footer` keep working.

## Where to Place Boundaries

Place error boundaries strategically around different parts of your app:

```tsx
function Dashboard() {
  return (
    <div className="grid grid-cols-3 gap-4">
      <ErrorBoundary fallback={<WidgetError name="Stats" />}>
        <StatsWidget />
      </ErrorBoundary>
      <ErrorBoundary fallback={<WidgetError name="Chart" />}>
        <ChartWidget />
      </ErrorBoundary>
      <ErrorBoundary fallback={<WidgetError name="Feed" />}>
        <FeedWidget />
      </ErrorBoundary>
    </div>
  );
}
```

Each widget is isolated. One failing widget does not take down the others.

## What Error Boundaries Cannot Catch

Error boundaries do not catch:

- Errors in event handlers (use try/catch instead)
- Errors in async code (use try/catch with setState)
- Errors in server-side rendering
- Errors thrown in the error boundary itself

For event handlers, use regular try/catch:

```tsx
function SaveButton() {
  const [error, setError] = useState<string | null>(null);

  const handleSave = async () => {
    try {
      await saveData();
      setError(null);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Save failed");
    }
  };

  return (
    <div>
      {error && <p className="text-red-500">{error}</p>}
      <button onClick={handleSave}>Save</button>
    </div>
  );
}
```

Error boundaries are one of those things you do not think about until you need them. But once you have seen an entire app go blank from one bug, you start wrapping everything.
