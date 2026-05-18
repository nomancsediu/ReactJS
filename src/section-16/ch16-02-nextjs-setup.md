# Next.js Setup

Setting up a Next.js project is surprisingly easy. The official tool does most of the work for you.

## Creating a New Project

Run this command and answer the prompts:

```bash
npx create-next-app@latest my-app
```

The prompts will ask you about TypeScript, ESLint, Tailwind CSS, the App Router, and more. I usually say yes to all of them. Here is what I pick:

```
TypeScript?          Yes
ESLint?              Yes
Tailwind CSS?        Yes
src/ directory?      Yes
App Router?          Yes
Import alias?        @/*
```

This gives you a solid starting point with everything configured.

## Project Structure

Once the project is created, here is what the important folders look like:

```
my-app/
  src/
    app/
      layout.tsx      # Root layout
      page.tsx        # Home page
      globals.css     # Global styles
    components/       # Your components
    lib/              # Utility functions
  public/             # Static assets
  next.config.ts      # Next.js config
  package.json
  tsconfig.json
```

The `app/` folder is where all the magic happens in the App Router. Every folder inside `app/` can become a route.

## Starting the Dev Server

```bash
npm run dev
```

This starts the development server at `http://localhost:3000`. It supports hot module replacement, so your changes show up instantly.

## Configuration

The `next.config.ts` file lets you customize Next.js. Here are some common settings I have used:

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // Allow images from external domains
  images: {
    domains: ["images.unsplash.com"],
  },
  // Redirect old routes
  async redirects() {
    return [
      {
        source: "/old-page",
        destination: "/new-page",
        permanent: true,
      },
    ];
  },
};

export default nextConfig;
```

## TypeScript Setup

Next.js comes with TypeScript already configured. The `tsconfig.json` has sensible defaults, including the `@/` path alias so you can write clean imports:

```ts
// Instead of this:
import Button from "../../../components/Button";

// You can write this:
import Button from "@/components/Button";
```

That is really all you need to get started. The default setup is good enough for most projects, and you can tweak things as you go. I spent way too long worrying about configuration before realizing I could just start building and adjust later.
