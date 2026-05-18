# React Server Components

React Server Components (RSC) are the biggest change to React since hooks. They let you render components on the server with zero JavaScript sent to the browser. I am still wrapping my head around them, but here is what I understand so far.

## The Concept

Traditional React sends component JavaScript to the browser, which then renders the HTML. Server Components flip this: the HTML is rendered on the server, and only the HTML reaches the browser. No JavaScript bundle for that component.

```tsx
// This is a Server Component (default in Next.js App Router)
async function BlogPosts() {
  const posts = await db.post.findMany();

  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

No `useEffect`. No loading spinner. No client-side JavaScript for fetching. The database query happens on the server, the HTML is generated, and the browser just displays it.

## What Stays on the Server

Server Components can do things that client components cannot:

- Access databases directly
- Read the file system
- Use server-only environment variables
- Keep large dependencies off the client bundle

```tsx
import { readFileSync } from "fs";
import { marked } from "marked"; // 50KB library, never sent to browser

async function MarkdownPage() {
  const content = readFileSync("./content.md", "utf-8");
  const html = await marked(content);

  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

The `marked` library is 50KB, but the browser never downloads it. Only the resulting HTML is sent.

## Streaming

Server Components support streaming, which means the browser can start displaying content before the entire page is ready:

```tsx
import { Suspense } from "react";

function Page() {
  return (
    <div>
      <h1>My Blog</h1>
      <Suspense fallback={<p>Loading posts...</p>}>
        <BlogPosts /> {/* This streams in when ready */}
      </Suspense>
      <Sidebar /> {/* This displays immediately */}
    </div>
  );
}
```

`Sidebar` appears right away. `BlogPosts` shows a fallback until the data is ready, then streams in.

## When to Use Server Components

Use server components when:

- The component fetches data from a database or API
- It renders static content that does not need interactivity
- It uses heavy dependencies you do not want in the client bundle
- SEO matters and you need HTML in the initial response

Use client components when:

- The component has interactivity (onClick, onChange)
- It uses React hooks that manage state
- It depends on browser-only APIs
- It needs to update in real time

## React 19 Features

React 19 adds more server component features:

- **Server Actions**: Functions that run on the server, callable from client components
- **use()**: A new hook to read promises and context directly in render

```tsx
// Server Action
"use server";
async function createPost(formData: FormData) {
  await db.post.create({
    data: { title: formData.get("title") as string },
  });
}
```

```tsx
// use() hook
import { use } from "react";

function PostList({ postsPromise }: { postsPromise: Promise<Post[]> }) {
  const posts = use(postsPromise);

  return (
    <ul>
      {posts.map((post) => <li key={post.id}>{post.title}</li>)}
    </ul>
  );
}
```

Server Components are still evolving, and the ecosystem is catching up. I am learning them gradually by building small projects with Next.js. The mental model is different from what I am used to, but the performance benefits are real.
