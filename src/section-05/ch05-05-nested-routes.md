# Nested Routes

As my apps grew, I noticed many pages shared the same layout. A sidebar stayed the same while the main content changed. Copying the layout into every page component was repetitive. Nested routes solved this beautifully.

## The Problem

Without nested routes, you repeat layout code:

```jsx
// Repeating layout in every component
function Dashboard() {
  return (
    <div className="flex">
      <Sidebar />
      <main><DashboardContent /></main>
    </div>
  );
}

function Settings() {
  return (
    <div className="flex">
      <Sidebar /> {/* Same sidebar, copied */}
      <main><SettingsForm /></main>
    </div>
  );
}
```

## The Solution: Nested Routes

Define child routes inside a parent route:

```jsx
<Routes>
  <Route path="/app" element={<AppLayout />}>
    <Route index element={<Dashboard />} />
    <Route path="settings" element={<Settings />} />
    <Route path="profile" element={<Profile />} />
  </Route>
</Routes>
```

The parent route renders `AppLayout`, and the child routes render inside it using `Outlet`.

## The Outlet Component

`Outlet` is a placeholder where the matched child route renders:

```jsx
function AppLayout() {
  return (
    <div className="flex">
      <Sidebar />
      <main className="flex-1 p-6">
        <Outlet />
      </main>
    </div>
  );
}
```

When you visit `/app/settings`, `AppLayout` renders with `Settings` inside the Outlet. The sidebar stays put, and only the main content changes.

## The Index Route

The `index` route is the default child. It renders when the parent path matches exactly:

```jsx
<Route path="/app" element={<AppLayout />}>
  <Route index element={<Dashboard />} />
  <Route path="settings" element={<Settings />} />
</Route>
```

Visiting `/app` renders `Dashboard` inside the Outlet. Without the index route, the Outlet would be empty.

## Relative Paths

Notice I wrote `path="settings"` not `path="/settings"`. Child routes use relative paths. They are automatically prefixed with the parent path.

```jsx
// These are equivalent
<Route path="/app" element={<AppLayout />}>
  <Route path="settings" element={<Settings />} />
</Route>

// Full path is /app/settings
```

No leading slash on child routes. This is different from v5, and I messed it up plenty of times before it stuck.

## Layout Routes

You can create routes that contribute layout without adding to the URL:

```jsx
<Route element={<AppLayout />}>
  <Route path="/dashboard" element={<Dashboard />} />
  <Route path="/settings" element={<Settings />} />
</Route>
```

This renders `AppLayout` with `Outlet` for both paths, but the URL is just `/dashboard` and `/settings`. No `/app` prefix.

## Deeply Nested Routes

Nesting can go as deep as you need:

```jsx
<Route path="/app" element={<AppLayout />}>
  <Route path="projects" element={<ProjectsLayout />}>
    <Route index element={<ProjectList />} />
    <Route path=":id" element={<ProjectDetail />} />
  </Route>
</Route>
```

`ProjectsLayout` has its own Outlet, and `ProjectDetail` renders inside it. The full path is `/app/projects/123`.

Nested routes keep layouts DRY and URLs logical. I use them in every project now.
