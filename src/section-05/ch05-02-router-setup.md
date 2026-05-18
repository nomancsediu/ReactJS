# Router Setup

Setting up React Router used to confuse me because there were so many options. Do I use BrowserRouter? HashRouter? MemoryRouter? Let me walk you through the simplest setup.

## Installing React Router

First, install the package:

```bash
npm install react-router-dom
```

That is it. The `react-router-dom` package includes everything you need for web apps. The core `react-router` package is for React Native and other environments.

## Basic Setup with BrowserRouter

Wrap your app with `BrowserRouter` at the top level. This gives all your components access to routing features.

```jsx
import { BrowserRouter } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </BrowserRouter>
  );
}
```

Three things are happening here:

1. **BrowserRouter** wraps everything and listens for URL changes
2. **Routes** is the container that matches the current URL to a route
3. **Route** defines a path and the component to render

## Alternative: CreateBrowserRouter

React Router v6.4 introduced a data router API that supports loaders and actions:

```jsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const router = createBrowserRouter([
  { path: '/', element: <Home /> },
  { path: '/about', element: <About /> },
  { path: '/contact', element: <Contact /> },
]);

function App() {
  return <RouterProvider router={router} />;
}
```

This approach is more powerful but also more complex. I recommend starting with `BrowserRouter` and switching to `createBrowserRouter` when you need data loading features.

## Where to Put the Router

I like to put the router in my main entry file:

```jsx
// main.jsx
import { BrowserRouter } from 'react-router-dom';
import App from './App';

createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

Then my `App.jsx` just has the routes:

```jsx
// App.jsx
function App() {
  return (
    <Routes>
      <Route path="/" element={<Home />} />
      <Route path="/about" element={<About />} />
    </Routes>
  );
}
```

This keeps the entry point clean and the routing logic in one place.

## Common Mistake

Forgetting to wrap with `BrowserRouter` gives you errors like "useNavigate may be used only in the context of a Router." Every component that uses routing hooks or components must be inside a router provider.

My setup is simple: wrap once at the top, define routes in App, and everything just works.
