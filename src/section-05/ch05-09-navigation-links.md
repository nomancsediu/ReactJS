# Navigation Links

Navigation in React apps needs special handling. Regular `<a>` tags cause full page reloads, which defeats the purpose of a single-page app. React Router provides components that navigate without reloading.

## Link Component

The `Link` component is the basic navigation element:

```jsx
import { Link } from 'react-router-dom';

function Navbar() {
  return (
    <nav>
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/blog">Blog</Link>
    </nav>
  );
}
```

`Link` renders an `<a>` tag under the hood but intercepts clicks to do client-side navigation. No page reload.

## NavLink for Active Styling

`NavLink` is like `Link` but knows when it is active. This is perfect for highlighting the current page in a navigation menu:

```jsx
import { NavLink } from 'react-router-dom';

function Navbar() {
  return (
    <nav className="flex gap-4">
      <NavLink
        to="/"
        className={({ isActive }) =>
          isActive ? 'text-blue-600 font-bold' : 'text-gray-600'
        }
      >
        Home
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) =>
          isActive ? 'text-blue-600 font-bold' : 'text-gray-600'
        }
      >
        About
      </NavLink>
    </nav>
  );
}
```

The `className` and `style` props accept a function that receives an `isActive` boolean. Use it to apply different styles for the current page.

## useLocation

The `useLocation` hook gives you the current location object:

```jsx
import { useLocation } from 'react-router-dom';

function Breadcrumbs() {
  const location = useLocation();
  const pathSegments = location.pathname.split('/').filter(Boolean);

  return (
    <div className="text-sm text-gray-500">
      <Link to="/">Home</Link>
      {pathSegments.map((segment, index) => {
        const path = '/' + pathSegments.slice(0, index + 1).join('/');
        return (
          <span key={path}>
            {' / '}
            <Link to={path}>{segment}</Link>
          </span>
        );
      })}
    </div>
  );
}
```

The location object contains `pathname`, `search`, `hash`, and `state`. It updates whenever the URL changes.

## useSearchParams

Query parameters are those `?key=value` things at the end of URLs. `useSearchParams` lets you read and modify them:

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get('category') || 'all';
  const sort = searchParams.get('sort') || 'newest';

  const handleCategoryChange = (newCategory) => {
    setSearchParams({ category: newCategory, sort });
  };

  return (
    <div>
      <select
        value={category}
        onChange={(e) => handleCategoryChange(e.target.value)}
      >
        <option value="all">All</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
      </select>
      <p>Showing: {category}, sorted by: {sort}</p>
    </div>
  );
}
```

The URL updates to `/products?category=electronics&sort=newest`, and you can bookmark or share it.

## Link vs NavLink

I use `Link` for general navigation and `NavLink` for menus and tabs where I need to show which item is active. If you do not need active state, `Link` is simpler.

## A Complete Navbar Example

```jsx
function Navbar() {
  return (
    <nav className="flex items-center gap-6 p-4 border-b">
      <NavLink to="/" end className={({ isActive }) =>
        isActive ? 'font-bold' : ''
      }>Home</NavLink>
      <NavLink to="/blog" className={({ isActive }) =>
        isActive ? 'font-bold' : ''
      }>Blog</NavLink>
      <NavLink to="/about" className={({ isActive }) =>
        isActive ? 'font-bold' : ''
      }>About</NavLink>
    </nav>
  );
}
```

The `end` prop on the home link ensures it is only active for exactly `/`, not `/blog` or `/about`.
