# SSR, SSG, and ISR

Next.js gives you multiple ways to render pages, and picking the right one makes a big difference. I used to think rendering was just "server or client," but it is more nuanced than that.

## Server-Side Rendering (SSR)

SSR generates the HTML on every request. The server fetches data, renders the page, and sends the finished HTML to the browser.

```tsx
// app/products/[id]/page.tsx
export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = await fetch(`https://api.example.com/products/${id}`, {
    cache: "no-store", // Always fetch fresh data
  }).then((res) => res.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}</p>
    </div>
  );
}
```

Use SSR when data changes frequently and must always be up to date, like a user dashboard or a shopping cart.

## Static Site Generation (SSG)

SSG generates the HTML at build time. The page is created once and served as a static file after that.

```tsx
// app/about/page.tsx
export default async function AboutPage() {
  const data = await fetch("https://api.example.com/about", {
    cache: "force-cache", // Cache at build time (default)
  }).then((res) => res.json());

  return <p>{data.description}</p>;
}
```

Use SSG when content rarely changes, like a blog post or a marketing page. It is the fastest option because there is no server work at request time.

## Incremental Static Regeneration (ISR)

ISR is the best of both worlds. It serves a static page but regenerates it in the background at a set interval.

```tsx
// app/blog/page.tsx
export default async function BlogPage() {
  const posts = await fetch("https://api.example.com/posts", {
    next: { revalidate: 60 }, // Revalidate every 60 seconds
  }).then((res) => res.json());

  return (
    <ul>
      {posts.map((post: { id: number; title: string }) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

Use ISR when content changes occasionally but does not need to be real-time, like a blog index or a product catalog.

## When to Use Each

| Strategy | Speed | Freshness | Best For |
|---|---|---|---|
| SSR | Slower | Always fresh | Dashboards, real-time data |
| SSG | Fastest | Stale until rebuild | Static content, docs |
| ISR | Fast | Fresh on interval | Blogs, catalogs, news |

Picking the right strategy took me a while to figure out. My rule of thumb now: start with SSG, move to ISR if data needs updating, and use SSR only when you truly need fresh data on every request.
