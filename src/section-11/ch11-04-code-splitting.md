# Code Splitting

Lazy loading individual components is great, but code splitting is the bigger picture. It is the strategy of breaking your JavaScript bundle into smaller pieces so the browser only downloads what it needs. Let me show you the different ways to split your code.

## Dynamic Imports

The foundation of code splitting is the dynamic `import()` syntax:

```jsx
// Static import: included in the main bundle
import { format } from "date-fns";

// Dynamic import: creates a separate chunk
const formatDate = async (date) => {
  const { format } = await import("date-fns");
  return format(date, "yyyy-MM-dd");
};
```

Static imports are resolved at build time. Dynamic imports create a separate chunk that loads at runtime. Use dynamic imports for code that is not needed immediately.

## Route-Based Splitting

The most impactful place to split is at route boundaries:

```jsx
// Each route gets its own chunk
const routes = [
  { path: "/", component: lazy(() => import("./pages/Home")) },
  { path: "/dashboard", component: lazy(() => import("./pages/Dashboard")) },
  { path: "/settings", component: lazy(() => import("./pages/Settings")) },
  { path: "/reports", component: lazy(() => import("./pages/Reports")) },
];
```

When a user visits `/dashboard`, only the Dashboard chunk loads. They never download the Settings or Reports code unless they navigate there.

## Feature-Based Splitting

Split by feature or user role. Admin features should not load for regular users:

```jsx
function App({ user }) {
  return (
    <div>
      <Navbar />
      <Routes>
        <Route path="/" element={<Home />} />
        {user.isAdmin && (
          <Route
            path="/admin"
            element={
              <Suspense fallback={<Loader />}>
                <AdminPanel /> {/* Only loads for admins */}
              </Suspense>
            }
          />
        )}
      </Routes>
    </div>
  );
}
```

## Vendor Splitting

Large libraries like charting or markdown parsers should be in their own chunks:

```jsx
// Vite does this automatically with manualChunks
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          "chart": ["recharts"],
          "markdown": ["marked", "highlight.js"],
          "ui-lib": ["@radix-ui/react-dialog", "@radix-ui/react-dropdown-menu"],
        },
      },
    },
  },
});
```

This keeps your main bundle small. The chart library only loads on pages that use charts.

## Prefetching

Load chunks in the background before the user needs them:

```jsx
// Prefetch on hover
function SettingsLink() {
  const handleMouseEnter = () => {
    import("./pages/Settings"); // Start loading early
  };

  return (
    <Link to="/settings" onMouseEnter={handleMouseEnter}>
      Settings
    </Link>
  );
}
```

When the user hovers over the link, the chunk starts loading. By the time they click, it is already cached. The navigation feels instant.

## How to Know What to Split

Run a bundle analyzer to see what takes up space:

```bash
# Install the analyzer
npm install --save-dev rollup-plugin-visualizer

# In vite.config.js
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [visualizer({ open: true })],
});
```

The visualizer generates a chart showing every module and its size. Target the largest chunks first. Splitting a 50KB library matters more than splitting a 2KB utility.

Code splitting is not about splitting everything. It is about sending the user only what they need right now. Split routes, split heavy features, split large libraries. Everything else can stay in the main bundle.
