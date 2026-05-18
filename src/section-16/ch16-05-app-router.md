# The App Router

The App Router is the newer routing system in Next.js, introduced in version 13. It replaced the Pages Router and brought some big changes. I found the transition confusing at first, but once I understood the concepts, everything clicked.

## App Router vs Pages Router

The Pages Router uses a `pages/` directory. The App Router uses an `app/` directory.

```
# Pages Router (old)
pages/
  index.tsx          → /
  about.tsx          → /about
  _app.tsx           # App wrapper

# App Router (new)
app/
  page.tsx           → /
  about/
    page.tsx         → /about
  layout.tsx         # Root layout
```

The App Router supports server components by default, nested layouts, and streaming. The Pages Router still works, but the App Router is where all the new features live.

## Layouts

Layouts wrap pages and persist across navigation. The root layout is required and must include `<html>` and `<body>` tags:

```tsx
// app/layout.tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        <nav>
          <a href="/">Home</a>
          <a href="/about">About</a>
        </nav>
        {children}
      </body>
    </html>
  );
}
```

Nested layouts are even cooler. Each folder can have its own layout:

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <aside>Sidebar</aside>
      <main>{children}</main>
    </div>
  );
}
```

When you navigate between `/dashboard/settings` and `/dashboard/profile`, the dashboard layout stays mounted. Only the `children` re-render.

## Templates

Templates work like layouts but remount on navigation:

```tsx
// app/dashboard/template.tsx
export default function DashboardTemplate({
  children,
}: {
  children: React.ReactNode;
}) {
  return <div>{children}</div>;
}
```

Use templates when you need to reset state or run effects on every navigation, like entering animations.

## Loading States

The `loading.tsx` file shows a fallback while the page loads:

```tsx
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return (
    <div className="animate-pulse">
      <div className="h-8 bg-gray-200 rounded w-1/3 mb-4" />
      <div className="h-4 bg-gray-200 rounded w-2/3" />
    </div>
  );
}
```

Next.js uses React Suspense under the hood. The loading UI appears instantly while the server fetches data, then swaps in the real content.

## Error States

Similarly, `error.tsx` catches runtime errors:

```tsx
// app/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

The App Router has a learning curve, but the layout system alone makes it worth it. I used to dread setting up shared layouts in React Router. Now it just works.
