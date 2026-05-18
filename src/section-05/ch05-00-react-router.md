# React Router

When I built my first React app, I had a problem. Everything lived on one page. Clicking a link reloaded the entire browser. It did not feel like a modern app at all.

That is when I discovered routing, specifically client-side routing. React Router is the library that makes single-page apps feel like multi-page websites. You get different views for different URLs, all without a single page reload.

## Why Routing Matters

Without routing, your React app is just one big view. Users cannot bookmark specific pages, cannot share links to specific content, and cannot use the browser back button. Routing fixes all of that.

## What You Will Learn

In this section, we will cover everything you need to know about React Router:

- **What routing is** and how SPA routing differs from traditional server routing
- **Setting up React Router** in your project
- **Configuring routes** with the Route component
- **Dynamic routes** with URL parameters
- **Nested routes** for shared layouts
- **Programmatic navigation** with useNavigate
- **Protected routes** for authenticated users
- **404 pages and error boundaries** for when things go wrong
- **Navigation links** with Link, NavLink, and friends

## A Quick Note

React Router has gone through many versions, and the API has changed significantly. We will use React Router v6, which is the current standard. If you see older tutorials with `Switch` or different route syntax, those are from v5 or earlier.

I spent a confusing week following outdated tutorials before I realized the version difference mattered. Make sure your installed version matches what you are learning.

Let us start navigating.
