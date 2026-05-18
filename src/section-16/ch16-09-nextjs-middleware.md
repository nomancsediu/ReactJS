# Next.js Middleware

Middleware in Next.js runs before a request is completed. It is perfect for checking things like authentication, redirecting users, or modifying requests. I think of it as a bouncer at the door of your app.

## The Middleware Function

Create a `middleware.ts` file at the root of your project (or inside `src/`):

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // This runs before the page loads
  console.log(`Request to: ${request.nextUrl.pathname}`);

  return NextResponse.next();
}
```

`NextResponse.next()` means "let the request continue." You can also return a redirect or a rewrite.

## Matching Paths

By default, middleware runs on every route. Use the `config` export to limit it:

```ts
export const config = {
  matcher: [
    "/dashboard/:path*",   // Only dashboard routes
    "/admin/:path*",       // Only admin routes
  ],
};
```

You can also use functions for more complex matching:

```ts
export const config = {
  matcher: [
    "/((?!api|_next/static|_next/image|favicon.ico).*)",
  ],
};
```

This runs middleware on everything except API routes and static files.

## Authentication Check

Here is a real example I use for protecting routes:

```ts
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const token = request.cookies.get("auth-token")?.value;

  // If no token and trying to access protected route
  if (!token && request.nextUrl.pathname.startsWith("/dashboard")) {
    const loginUrl = new URL("/login", request.url);
    loginUrl.searchParams.set("from", request.nextUrl.pathname);
    return NextResponse.redirect(loginUrl);
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

If the user has no auth token, they get redirected to the login page. The `from` parameter lets the login page redirect back after signing in.

## Redirects

You can redirect users to a different URL:

```ts
export function middleware(request: NextRequest) {
  // Redirect old URLs to new ones
  if (request.nextUrl.pathname === "/old-blog") {
    return NextResponse.redirect(new URL("/blog", request.url));
  }

  return NextResponse.next();
}
```

## Rewrites

Rewrites show different content without changing the URL in the browser:

```ts
export function middleware(request: NextRequest) {
  // Show /blog content at /articles URL
  if (request.nextUrl.pathname.startsWith("/articles")) {
    return NextResponse.rewrite(new URL("/blog", request.url));
  }

  return NextResponse.next();
}
```

## Setting Headers

You can add or modify request headers:

```ts
export function middleware(request: NextRequest) {
  const response = NextResponse.next();
  response.headers.set("x-custom-header", "my-value");
  return response;
}
```

Middleware runs on the Edge runtime, which means it is fast but has some limitations. You cannot use Node.js APIs like `fs` or access databases directly. Keep it lightweight and do the heavy lifting in your route handlers or server components instead.
