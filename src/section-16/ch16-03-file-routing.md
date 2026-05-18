# File-Based Routing

One of my favorite things about Next.js is that routing is based on your file structure. You do not need to configure routes manually. Just create folders and files, and Next.js figures out the rest.

## How It Works

In the App Router, every folder inside `app/` maps to a URL segment. Every `page.tsx` file makes that route accessible.

```
app/
  page.tsx              → /
  about/
    page.tsx            → /about
  blog/
    page.tsx            → /blog
    [slug]/
      page.tsx          → /blog/my-post
```

## Basic Pages

A page is just a component exported from `page.tsx`:

```tsx
// app/about/page.tsx
export default function AboutPage() {
  return (
    <div>
      <h1>About Me</h1>
      <p>I am learning Next.js and writing about it.</p>
    </div>
  );
}
```

Visit `/about` and this page renders. No router setup needed.

## Dynamic Routes

Use square brackets for dynamic segments:

```tsx
// app/blog/[slug]/page.tsx
export default async function BlogPost({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;

  return <h1>Post: {slug}</h1>;
}
```

Visit `/blog/hello-world` and `slug` will be `"hello-world"`.

## Catch-All Routes

Use `[...slug]` to catch all segments after a point:

```tsx
// app/docs/[...slug]/page.tsx
export default async function DocsPage({
  params,
}: {
  params: Promise<{ slug: string[] }>;
}) {
  const { slug } = await params;

  return <p>Path: {slug.join("/")}</p>;
}
```

- `/docs/getting-started` shows "Path: getting-started"
- `/docs/api/reference/hooks` shows "Path: api/reference/hooks"

## Optional Catch-All

Add another set of brackets for an optional catch-all: `[[...slug]]`. This also matches the parent route without any segments.

- `/docs` matches (unlike `[...slug]`)
- `/docs/a/b` also matches

## Layouts and Nested Routing

Each folder can have a `layout.tsx` that wraps all pages inside it:

```tsx
// app/blog/layout.tsx
export default function BlogLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="max-w-2xl mx-auto">
      <h2>My Blog</h2>
      {children}
    </div>
  );
}
```

Every page under `/blog` gets wrapped with this layout. The root `app/layout.tsx` wraps everything.

File-based routing felt strange at first because I was used to React Router. But once I got the hang of it, I found it much simpler. Less configuration means fewer bugs and faster development.
