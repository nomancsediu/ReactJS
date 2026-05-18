# Route Configuration

Once your router is set up, you need to tell it which URLs map to which components. That is route configuration, and it is more nuanced than I first thought.

## The Route Component

Each `Route` needs two things: a `path` and an `element`.

```jsx
<Routes>
  <Route path="/" element={<Home />} />
  <Route path="/about" element={<About />} />
  <Route path="/blog" element={<Blog />} />
</Routes>
```

When the URL matches a path, React Router renders the corresponding element. Simple enough.

## The Routes Wrapper

The `Routes` component is smarter than it looks. It looks at the current URL and finds the best matching route, not just the first one.

```jsx
<Routes>
  <Route path="/users" element={<UserList />} />
  <Route path="/users/new" element={<NewUser />} />
</Routes>
```

If you visit `/users/new`, React Router renders `NewUser`, not `UserList`. It picks the most specific match. In older versions of React Router, you had to worry about route order. Not anymore.

## Path Matching

Paths can be exact or flexible:

```jsx
// Exact match: only /about
<Route path="/about" element={<About />} />

// Trailing slashes: /about/ also matches
// Case sensitivity: paths are case-sensitive by default
```

React Router v6 matches exactly by default. You do not need an `exact` prop like in v5.

## Matching Behavior

The matching algorithm uses a scoring system. More specific paths score higher:

```jsx
<Routes>
  {/* Score: lower - matches /users */}
  <Route path="/users" element={<UserList />} />

  {/* Score: higher - matches /users/new specifically */}
  <Route path="/users/new" element={<NewUser />} />

  {/* Score: higher - matches /users/:id with a parameter */}
  <Route path="/users/:id" element={<UserProfile />} />
</Routes>
```

Visiting `/users/123` matches the `:id` route. Visiting `/users/new` matches the `/users/new` route because static segments rank higher than dynamic ones.

## Multiple Routes

You can have multiple `Routes` components in your app. They work independently:

```jsx
function App() {
  return (
    <div>
      <Sidebar>
        <Routes>
          <Route path="/" element={<SidebarHome />} />
          <Route path="/about" element={<SidebarAbout />} />
        </Routes>
      </Sidebar>
      <Main>
        <Routes>
          <Route path="/" element={<MainHome />} />
          <Route path="/about" element={<MainAbout />} />
        </Routes>
      </Main>
    </div>
  );
}
```

## Optional: Route Object Configuration

You can also define routes as objects instead of JSX:

```jsx
const routes = [
  { path: '/', element: <Home /> },
  { path: '/about', element: <About /> },
  { path: '/blog', element: <Blog /> },
];

function App() {
  return useRoutes(routes);
}
```

This is handy for large apps where you want to organize routes in separate files.
