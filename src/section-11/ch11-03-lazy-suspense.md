# Lazy Loading with React.lazy and Suspense

When your app grows, your JavaScript bundle grows with it. Every component, every library, every utility gets packed into one big file. The browser has to download and parse all of it before showing anything. That is slow.

Lazy loading fixes this by loading code only when it is needed. Instead of shipping everything upfront, you split your app into chunks and load them on demand.

## React.lazy

`React.lazy` lets you load a component dynamically:

```jsx
import { lazy, Suspense } from "react";

// Instead of this (loaded immediately):
// import HeavyChart from "./HeavyChart";

// Do this (loaded when first rendered):
const HeavyChart = lazy(() => import("./HeavyChart"));
```

`React.lazy` takes a function that returns a dynamic `import()`. That import creates a separate chunk that loads only when the component renders.

## Suspense for Fallback UI

Lazy components need a Suspense boundary to show something while loading:

```jsx
function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart />
      </Suspense>
    </div>
  );
}

function ChartSkeleton() {
  return (
    <div className="animate-pulse">
      <div className="h-64 bg-gray-200 rounded" />
    </div>
  );
}
```

When `HeavyChart` is first rendered, React shows the fallback. When the chunk finishes loading, React swaps in the real component.

## Lazy Loading Routes

The biggest win is lazy loading route components. Users who visit the home page do not need the settings page code:

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route } from "react-router-dom";

const Home = lazy(() => import("./pages/Home"));
const Settings = lazy(() => import("./pages/Settings"));
const Profile = lazy(() => import("./pages/Profile"));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/settings" element={<Settings />} />
          <Route path="/profile" element={<Profile />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

Each page becomes its own chunk. The initial load only includes the home page code plus the router. The rest loads when the user navigates.

## Named Exports

`React.lazy` works with default exports. If your component uses a named export, wrap it:

```jsx
// Component uses: export function Settings() { ... }
const Settings = lazy(() =>
  import("./pages/Settings").then((module) => ({
    default: module.Settings,
  }))
);
```

## Nested Suspense

You can nest Suspense boundaries for granular control:

```jsx
<Suspense fallback={<PageLoader />}>
  <Layout>
    <Suspense fallback={<SidebarSkeleton />}>
      <Sidebar />
    </Suspense>
    <Suspense fallback={<ContentSkeleton />}>
      <MainContent />
    </Suspense>
  </Layout>
</Suspense>
```

Each section loads independently. The sidebar can appear before the main content finishes, giving users a sense of progress.

Lazy loading is one of the easiest performance wins in React. Add it to your routes first, then lazy load heavy components. Your users will see faster initial loads immediately.
