# Environment Variables in React

Environment variables are how you keep secrets out of your code and configure your app for different environments. I used to hardcode API URLs and API keys directly in my components. That worked until I deployed and everything broke because the URLs were wrong. Never again.

## The VITE_ Prefix

Vite only exposes environment variables that start with `VITE_` to your client-side code. This is a security feature. Any variable without this prefix stays on the server and is never bundled into your JavaScript.

```bash
# .env
VITE_API_URL=https://api.myapp.com
VITE_APP_TITLE=ReactJS Mastery

# This will NOT be available in your React code
SECRET_KEY=something_secret
```

Access them in your code with `import.meta.env`:

```jsx
function App() {
  return (
    <div>
      <h1>{import.meta.env.VITE_APP_TITLE}</h1>
      <p>API URL: {import.meta.env.VITE_API_URL}</p>
    </div>
  );
}
```

## .env Files Per Environment

Vite loads environment files in a specific order. I use separate files for each environment:

```bash
# .env                 - loaded in all environments
VITE_APP_NAME=MyApp

# .env.development     - loaded during development (vite dev)
VITE_API_URL=http://localhost:3001/api

# .env.production      - loaded during production build (vite build)
VITE_API_URL=https://api.myapp.com

# .env.local           - loaded in all environments, git-ignored
VITE_DEBUG=true
```

The `.env.local` file is perfect for personal overrides. It is git-ignored by default, so each developer can have their own settings without affecting the team.

## Runtime vs Build-Time Variables

Here is something that tripped me up: Vite replaces environment variables at build time, not runtime. That means once you build your app, the values are baked into the JavaScript bundle.

If you need runtime configuration (values that change after deployment), you need a different approach:

```html
<!-- public/index.html -->
<script>
  window.__APP_CONFIG__ = {
    apiUrl: '%VITE_API_URL%',
  };
</script>
```

Or load a config file at runtime:

```jsx
// Load runtime config from a JSON file
async function loadConfig() {
  const response = await fetch('/config.json');
  const config = await response.json();
  return config;
}
```

## Type Safety for Env Variables

I like to create a typed helper so I do not misspell variable names:

```ts
// src/lib/env.ts
interface EnvVariables {
  VITE_API_URL: string;
  VITE_APP_TITLE: string;
  VITE_FIREBASE_API_KEY: string;
}

export const env: EnvVariables = {
  VITE_API_URL: import.meta.env.VITE_API_URL || '/api',
  VITE_APP_TITLE: import.meta.env.VITE_APP_TITLE || 'MyApp',
  VITE_FIREBASE_API_KEY: import.meta.env.VITE_FIREBASE_API_KEY || '',
};
```

## Never Commit Secrets

I repeat: never put API keys or secrets in `.env` files that get committed to git. Even with the `VITE_` prefix, these values end up in the browser bundle. Any value that is truly secret must stay on the backend. The frontend should only contain configuration that is safe for users to see.
