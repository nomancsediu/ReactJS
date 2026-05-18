# Data Fetching in Next.js

Data fetching in Next.js is simpler than I expected. You just use `fetch` in server components, and Next.js handles the rest. But the caching options took me a while to understand, so let me break them down.

## Fetch in Server Components

In a server component, you can use `fetch` directly at the top level. No `useEffect`, no loading state, no client-side fetching:

```tsx
// app/posts/page.tsx
export default async function PostsPage() {
  const res = await fetch("https://api.example.com/posts");
  const posts = await res.json();

  return (
    <ul>
      {posts.map((post: { id: number; title: string }) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

The data is fetched on the server before the HTML is sent to the browser. The user never sees a loading spinner.

## Caching Options

Next.js extends the `fetch` API with caching and revalidation options.

### Force Cache (Default in Some Cases)

Caches the response and reuses it:

```tsx
const res = await fetch("https://api.example.com/posts", {
  cache: "force-cache",
});
```

### No Store

Always fetches fresh data, never caches:

```tsx
const res = await fetch("https://api.example.com/posts", {
  cache: "no-store",
});
```

Use this for real-time data like user dashboards or live scores.

### Revalidate

Caches the response but revalidates at an interval (in seconds):

```tsx
const res = await fetch("https://api.example.com/posts", {
  next: { revalidate: 60 }, // Revalidate every 60 seconds
});
```

This is ISR. The cached version is served instantly, and Next.js regenerates in the background.

## Parallel Data Fetching

When a page needs multiple data sources, fetch them in parallel to avoid waterfalls:

```tsx
export default async function DashboardPage() {
  // These run in parallel
  const [userRes, postsRes, statsRes] = await Promise.all([
    fetch("https://api.example.com/user"),
    fetch("https://api.example.com/posts"),
    fetch("https://api.example.com/stats"),
  ]);

  const [user, posts, stats] = await Promise.all([
    userRes.json(),
    postsRes.json(),
    statsRes.json(),
  ]);

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <p>You have {posts.length} posts</p>
      <p>Views today: {stats.views}</p>
    </div>
  );
}
```

## On-Demand Revalidation

You can trigger revalidation from an API route or server action:

```ts
// app/api/revalidate/route.ts
import { revalidatePath } from "next/cache";
import { NextResponse } from "next/server";

export async function POST() {
  revalidatePath("/posts"); // Clear cache for /posts
  return NextResponse.json({ revalidated: true });
}
```

There is also `revalidateTag` for more targeted cache invalidation:

```tsx
// Fetch with a tag
const res = await fetch("https://api.example.com/posts", {
  next: { tags: ["posts"] },
});
```

```ts
// Invalidate by tag
import { revalidateTag } from "next/cache";
revalidateTag("posts");
```

My advice: start with the defaults and add caching options only when you need them. Premature optimization with caching just makes things harder to debug.
