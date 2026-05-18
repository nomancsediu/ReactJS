# Server and Client Components

This is the topic that confused me the most when I started learning Next.js. The idea that some components run on the server and some on the client felt weird at first. But once it clicked, it made total sense.

## The Default: Server Components

In the App Router, every component is a server component by default. That means it runs on the server and never ships JavaScript to the browser.

```tsx
// This is a server component (no directive needed)
async function PostList() {
  const posts = await fetch("https://api.example.com/posts").then((res) =>
    res.json()
  );

  return (
    <ul>
      {posts.map((post: { id: number; title: string }) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

No `useEffect`. No loading state. Just fetch the data and render it. The browser receives only the HTML.

## Adding "use client"

When you need interactivity, you opt into client components with the `"use client"` directive:

```tsx
"use client";

import { useState } from "react";

export default function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

You need `"use client"` when your component uses:

- `useState`, `useEffect`, `useReducer`, or other hooks with state
- Event handlers like `onClick` or `onChange`
- Browser-only APIs like `window` or `document`
- A client-side library like a chart or map component

## Component Boundaries

The boundary between server and client matters. Server components can import and render client components. But client components cannot import server components.

```tsx
// This works: Server component renders a client component
import Counter from "./Counter"; // Client component

export default function Page() {
  return (
    <div>
      <h1>My Page</h1>
      <Counter /> {/* Client component here */}
    </div>
  );
}
```

```tsx
// This does NOT work: Client component importing a server component
"use client";
import ServerData from "./ServerData"; // This breaks!

export default function Wrapper() {
  return <ServerData />;
}
```

The workaround is to pass server components as children:

```tsx
// page.tsx (server component)
import ClientWrapper from "./ClientWrapper";
import ServerChild from "./ServerChild";

export default function Page() {
  return (
    <ClientWrapper>
      <ServerChild /> {/* Passed as children, works fine */}
    </ClientWrapper>
  );
}
```

## "use server"

The `"use server"` directive marks server actions, which are functions that run on the server but can be called from client components:

```tsx
// actions.ts
"use server";

export async function createPost(formData: FormData) {
  const title = formData.get("title");
  // Save to database...
}
```

```tsx
// form.tsx
"use client";

import { createPost } from "./actions";

export default function PostForm() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

My biggest takeaway: keep components as server components unless they absolutely need to be client components. Less JavaScript shipped to the browser means faster pages.
