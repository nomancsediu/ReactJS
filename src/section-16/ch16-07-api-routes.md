# API Routes in Next.js

Next.js lets you build API endpoints right inside your app. No separate backend needed for simple use cases. I love this because it keeps everything in one place.

## Route Handlers

In the App Router, API routes are defined using `route.ts` files inside your `app/` directory:

```
app/
  api/
    hello/
      route.ts      → /api/hello
    users/
      route.ts      → /api/users
```

## Handling Requests

Export named functions for each HTTP method:

```ts
// app/api/hello/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({ message: "Hello from the API!" });
}

export async function POST(request: Request) {
  const body = await request.json();
  return NextResponse.json(
    { received: body },
    { status: 201 }
  );
}
```

## Reading Request Data

You can access URL parameters, query strings, and the request body:

```ts
// app/api/users/[id]/route.ts
import { NextResponse } from "next/server";

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;

  // Read query parameters
  const { searchParams } = new URL(request.url);
  const format = searchParams.get("format");

  // Fetch user from database...
  const user = { id, name: "Alice", format };

  return NextResponse.json(user);
}
```

## Building a REST API

Here is a simple CRUD example:

```ts
// app/api/posts/route.ts
import { NextResponse } from "next/server";

const posts = [
  { id: 1, title: "Learning Next.js" },
  { id: 2, title: "React Server Components" },
];

export async function GET() {
  return NextResponse.json(posts);
}

export async function POST(request: Request) {
  const body = await request.json();
  const newPost = { id: posts.length + 1, title: body.title };
  posts.push(newPost);
  return NextResponse.json(newPost, { status: 201 });
}
```

```ts
// app/api/posts/[id]/route.ts
import { NextResponse } from "next/server";

export async function DELETE(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  // Delete post logic here...
  return NextResponse.json({ deleted: id });
}
```

## Middleware for APIs

You can add middleware to check things like authentication before your route handler runs. We will cover middleware in detail in the next chapter, but here is a quick idea:

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.headers.get("authorization");

  if (!token) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  return NextResponse.next();
}

export const config = {
  matcher: "/api/:path*",
};
```

API routes in Next.js are not meant to replace a full backend, but they are perfect for prototyping, small apps, and handling form submissions. I use them all the time for quick projects.
