# Bundle Optimization

Your JavaScript bundle is everything your user has to download before your app works. A big bundle means slow load times, especially on mobile networks. I was shocked when I first analyzed my bundle and found it was 2MB. Let me show you how to shrink it.

## Analyzing Your Bundle

First, see what is in there:

```bash
# Install the visualizer
npm install --save-dev rollup-plugin-visualizer
```

```js
// vite.config.js
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [
    visualizer({
      filename: "./bundle-stats.html",
      open: true,
    }),
  ],
});
```

Run your build, open the generated HTML file, and you will see a treemap. Each rectangle is a module. Bigger rectangles are bigger chunks of your bundle. You will quickly spot the heavy modules.

## Tree Shaking

Tree shaking removes unused exports from your bundle. It works automatically with ES modules:

```jsx
// utils.js exports 10 functions
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) { return a / b; }
// ... 7 more

// Your code only imports one
import { add } from "./utils";
// The other 9 are shaken out of the bundle
```

Tree shaking works when you use named imports. It fails with some patterns:

```jsx
// Tree shaking works:
import { format } from "date-fns";

// Tree shaking cannot work:
import _ from "lodash"; // Imports everything
_.chunk(array, 2);

// Fix: import only what you need
import chunk from "lodash/chunk";
```

Check your bundle analyzer for libraries that are imported entirely. Those are easy wins.

## Vendor Splitting

Separate third-party code from your own code. Vendor libraries change less often, so they can be cached longer:

```js
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          router: ["react-router-dom"],
          charts: ["recharts"],
        },
      },
    },
  },
});
```

When you update your app code, the vendor chunk stays cached in the browser. Users only re-download what changed.

## Production Builds

Always build for production when measuring performance:

```bash
# Development builds are unminified and include extra warnings
npm run build

# Production builds are minified, tree-shaken, and optimized
```

Vite handles minification, tree shaking, and chunk splitting automatically in production mode. Never judge performance from a dev server.

## Lighthouse Audits

Lighthouse gives you a comprehensive performance score:

```bash
# Run from Chrome DevTools: Lighthouse tab
# Or from command line:
npx lighthouse http://localhost:3000 --output html --output-path ./lighthouse-report.html
```

Lighthouse checks:

- **First Contentful Paint** when the first content appears
- **Largest Contentful Paint** when the main content is visible
- **Total Blocking Time** how long the main thread is blocked
- **Cumulative Layout Shift** how much the page jumps around

Aim for a performance score above 90. The report tells you exactly what to fix.

## Quick Wins Checklist

- Replace full library imports with specific imports (`lodash/get` instead of `lodash`)
- Lazy load routes and heavy components
- Compress images and use WebP
- Enable gzip or Brotli on your server
- Use the bundle analyzer to find surprise dependencies
- Keep your vendor chunks separate for better caching

Bundle optimization is ongoing work. Every new dependency adds weight. Check your bundle size regularly and question every large import. Your users on slow connections will thank you.
