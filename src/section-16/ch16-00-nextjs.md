# Moving Beyond Client-Side React

So far in this book, we have built React apps that run entirely in the browser. The server just sends an empty HTML page, and JavaScript does all the work. That approach works, but it has limits.

Pages load slower because the browser must download and run JavaScript before showing anything. Search engines struggle to index your content because there is no HTML to read. And some features, like reading from a database securely, just cannot happen on the client.

This is where Next.js comes in.

Next.js is a framework built on top of React. It adds server-side capabilities that plain React does not have out of the box. With Next.js, you can render pages on the server before sending them to the browser. You get faster initial loads, better SEO, and a smoother user experience.

In this section, I will walk through what I have been learning about Next.js. We will cover:

- What Next.js is and why it exists
- Setting up a new Next.js project
- File-based routing and how it works
- Different rendering strategies like SSR, SSG, and ISR
- The App Router and how it changed things
- Server components versus client components
- Building API routes inside Next.js
- Data fetching patterns and caching
- Middleware for auth and redirects
- Deploying your Next.js app

I am still learning all of this myself, so I will keep things simple and practical. No jargon without explanation. No skipping steps. Just honest notes from someone figuring it out as they go.

If you have been comfortable with React but curious about what comes next (pun intended), this section is for you. Let us dive in.
