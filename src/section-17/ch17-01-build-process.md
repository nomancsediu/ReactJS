# The Build Process

Before you deploy a React app, you need to build it. The build process takes your source code and turns it into optimized files that a browser can run efficiently.

## Running the Build

```bash
npm run build
```

This command does several things:

1. **Compiles TypeScript** to JavaScript
2. **Bundles** all your modules into fewer files
3. **Minifies** the code to reduce file size
4. **Tree-shakes** to remove unused code
5. **Hashes filenames** for cache busting

## What Gets Generated

After running the build, you get an output folder. For a Vite project, it is `dist/`. For Create React App or Next.js, it is `build/` or `.next/`.

```
dist/
  index.html              # Entry point
  assets/
    index-3f2a1b.js       # Bundled JavaScript
    index-8c4d2e.css      # Bundled CSS
    logo-a1b2c3.svg       # Static assets
```

The hash in the filename changes whenever the file content changes. This ensures browsers download new files instead of using cached old ones.

## What Actually Happens

Let me walk through what the build tool (usually Vite or Webpack) does:

```jsx
// Your source code (multiple files)
import React from "react";
import { useState } from "react";
import "./styles.css";
import logo from "./logo.svg";

function App() {
  const [count, setCount] = useState(0);
  return (
    <div className="app">
      <img src={logo} alt="Logo" />
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

The build tool:

1. Resolves all imports and bundles them together
2. Removes `import` statements (browsers do not understand them)
3. Converts JSX to `React.createElement` calls
4. Minifies variable names and removes whitespace
5. Extracts CSS into separate files

## Checking the Output

```bash
# Preview the build locally
npm run preview
```

This starts a local server with your built files so you can test the production version before deploying.

## Build Optimization Flags

In Vite, you can customize the build:

```ts
// vite.config.ts
export default defineConfig({
  build: {
    outDir: "dist",
    sourcemap: true,       // Generate source maps
    minify: "terser",      // Use Terser for minification
    chunkSizeWarningLimit: 1000,
  },
});
```

## Environment Variables at Build Time

Variables prefixed with `VITE_` are embedded at build time:

```bash
VITE_API_URL=https://api.example.com npm run build
```

```tsx
const apiUrl = import.meta.env.VITE_API_URL;
```

The value is literally replaced in the built JavaScript. You cannot change it after the build without rebuilding.

Understanding the build process helped me debug deployment issues. When something breaks in production but works locally, the build output is usually where I start looking.
