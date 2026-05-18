# Tailwind CSS Setup

Tailwind CSS has become the most popular way to style React apps, and for good reason. It gives you utility classes that let you build designs directly in your markup. No context switching between CSS files and components.

## What Is Tailwind?

Tailwind is a utility-first CSS framework. Instead of writing custom CSS classes, you compose designs using small, single-purpose classes:

```jsx
<button className="bg-green-600 text-white px-4 py-2 rounded-lg hover:bg-green-700">
  Save
</button>
```

At first, the long class names look messy. But once you get used to it, it is incredibly fast. You style components without ever leaving your JSX.

## Installing Tailwind with Vite

```bash
npm install tailwindcss @tailwindcss/vite
```

## Vite Configuration

Add the Tailwind plugin to your Vite config:

```javascript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

## Importing Tailwind

Add the Tailwind import to your main CSS file:

```css
/* src/index.css */
@import "tailwindcss";
```

That is it. Tailwind is now available in your entire project.

## PostCSS Configuration

If your project uses PostCSS directly (some older setups do), you may need a `postcss.config.js`:

```javascript
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

With the Vite plugin approach above, this is usually not needed. The plugin handles everything.

## Verifying the Setup

Create a simple component to test that Tailwind is working:

```jsx
function App() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <div className="bg-white p-8 rounded-xl shadow-lg">
        <h1 className="text-2xl font-bold text-gray-800">
          Tailwind is working!
        </h1>
        <p className="text-gray-600 mt-2">
          If you can see styled content, setup is complete.
        </p>
      </div>
    </div>
  );
}
```

## Common Setup Mistakes

I ran into these when I first set up Tailwind:

- **Forgetting the CSS import.** Without `@import "tailwindcss"`, none of the classes work.
- **Wrong plugin order.** The Tailwind Vite plugin should come after other plugins.
- **Old installation guides.** Tailwind v4 changed the setup process significantly. Make sure you follow v4 documentation, not v3.

With Tailwind installed, we can start building real components with utility classes in the next chapter.
