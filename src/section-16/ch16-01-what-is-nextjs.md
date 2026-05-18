# What Next.js Is and Why It Exists

Next.js is a React framework created by Vercel. It adds server-side features to React that you would otherwise have to build yourself. Think of it as React with superpowers.

## Why Next.js Exists

Plain React is a client-side library. The server sends a nearly empty HTML file, and React takes over in the browser. This works fine for many apps, but it creates problems:

- **Slow first paint** because the browser waits for JavaScript before rendering
- **Poor SEO** because search engine bots see an empty page
- **No server logic** because everything runs in the browser

Next.js solves these by letting you render React on the server.

## React vs Next.js

Here is a quick comparison:

| Feature | React | Next.js |
|---|---|---|
| Rendering | Client only | Server and client |
| Routing | Manual (React Router) | File-based |
| API routes | Separate backend needed | Built in |
| SSR | Not built in | Built in |
| SSG | Not built in | Built in |
| Image optimization | Manual | Automatic |

## When to Use Next.js

I would reach for Next.js when:

- SEO matters (blogs, e-commerce, marketing sites)
- You need fast initial page loads
- You want a full-stack app with API routes
- You need server-side data fetching

I would stick with plain React (like Vite + React) when:

- The app is behind a login (SEO does not matter)
- It is a highly interactive dashboard
- You already have a separate backend
- The project is small and simple

```jsx
// Plain React: client-side only
function App() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data')
      .then(res => res.json())
      .then(setData);
  }, []);

  if (!data) return <p>Loading...</p>;
  return <p>{data.message}</p>;
}
```

```jsx
// Next.js: server-side rendering
async function Page() {
  const res = await fetch('https://api.example.com/data');
  const data = await res.json();

  return <p>{data.message}</p>;
}
```

Notice how the Next.js version does not need `useEffect` or loading state. The data is fetched on the server before the page reaches the browser.

Next.js is not a replacement for React. It is React plus a server. That distinction matters, and it took me a while to really understand it.
