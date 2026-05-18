# Environment Management

Managing environment variables across different stages of your app is one of those things that seems simple until you mess it up. I have pushed secrets to GitHub more than once, and it is never fun. Let me share what I have learned.

## Environment Stages

Most apps have at least three environments:

- **Development**: Your local machine
- **Staging/Preview**: For testing before release
- **Production**: The live site

Each environment might connect to different databases, use different API URLs, or enable different features.

## .env Files

Use `.env` files to manage variables locally:

```bash
# .env (shared defaults)
VITE_APP_NAME=ReactJS Mastery

# .env.development (local dev)
VITE_API_URL=http://localhost:3001
VITE_ENABLE_MOCK=true

# .env.production (production build)
VITE_API_URL=https://api.example.com
VITE_ENABLE_MOCK=false
```

Vite automatically loads the right file based on the mode. Next.js does the same with `NEXT_PUBLIC_` prefixed variables.

**Important**: Never commit `.env.local` or files with real secrets. Add them to `.gitignore`:

```gitignore
.env.local
.env.*.local
```

## Build-Time vs Runtime Variables

This distinction tripped me up for a while.

**Build-time variables** are baked into the JavaScript bundle at build time:

```tsx
// Vite
const apiUrl = import.meta.env.VITE_API_URL;

// Next.js (exposed to browser)
const apiUrl = process.env.NEXT_PUBLIC_API_URL;
```

You cannot change these without rebuilding. If the API URL changes, you must run `npm run build` again.

**Runtime variables** are read when the app starts on the server:

```tsx
// Next.js server-only (not exposed to browser)
const dbUrl = process.env.DATABASE_URL;
```

These can be changed without rebuilding. Just restart the server with the new value.

## Secrets

Never put secrets in client-side code. Any variable prefixed with `VITE_` or `NEXT_PUBLIC_` is visible in the browser.

```tsx
// NEVER do this
VITE_SECRET_KEY=abc123    // Anyone can see this in the bundle!

// Do this instead (Next.js server-only)
SECRET_KEY=abc123          // Only on the server

// Access it in a server component or API route
const secret = process.env.SECRET_KEY;
```

## Managing Secrets in Production

On Vercel and Netlify, set secrets in the dashboard. They never appear in your code.

For Docker, pass them as environment variables:

```bash
docker run -e DATABASE_URL=postgresql://... -e SECRET_KEY=abc123 my-app
```

Or use a `.env` file:

```bash
docker run --env-file .env.production my-app
```

## Validation with Zod

Validate environment variables at startup so you catch missing values early:

```ts
// lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  VITE_API_URL: z.string().url(),
  VITE_APP_NAME: z.string().min(1),
});

export const env = envSchema.parse(import.meta.env);
```

If a required variable is missing, the app fails immediately with a clear error instead of crashing later in some weird way.

My rule now: if a variable is secret, it stays on the server. If it is in the browser bundle, assume everyone can see it. That simple rule has saved me from a lot of headaches.
