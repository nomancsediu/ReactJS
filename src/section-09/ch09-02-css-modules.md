# CSS Modules

CSS Modules give you locally scoped CSS with zero runtime cost. No class name conflicts, no specificity wars. It is one of my favorite approaches for medium-sized projects.

## What Are CSS Modules?

A CSS Module is a regular CSS file where all class names and animation names are scoped locally by default. When you import a CSS Module, you get an object with the original class names as keys and unique generated names as values.

## Setting Up CSS Modules

CSS Modules work out of the box with Vite. Just name your files with the `.module.css` extension:

```
src/
  components/
    Button/
      Button.tsx
      Button.module.css
```

## Writing a CSS Module

```css
/* Button.module.css */
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
}

.primary {
  background-color: #16a34a;
  color: white;
}

.secondary {
  background-color: #e5e7eb;
  color: #374151;
}
```

## Using the Module in a Component

```tsx
import styles from "./Button.module.css";

function Button({ variant, children }) {
  return (
    <button className={`${styles.button} ${styles[variant]}`}>
      {children}
    </button>
  );
}

// Usage
<Button variant="primary">Save</Button>
<Button variant="secondary">Cancel</Button>
```

At build time, Vite transforms the class names into something like `Button_button_1a2b3` and `Button_primary_4c5d6`. No two components will ever have conflicting class names.

## Composition

CSS Modules support composition, which lets you build on existing classes:

```css
/* Card.module.css */
.base {
  padding: 16px;
  border-radius: 8px;
  background-color: white;
}

.elevated {
  composes: base;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.bordered {
  composes: base;
  border: 1px solid #d1d5db;
}
```

The `composes` keyword includes all properties from the referenced class. This keeps your CSS DRY without preprocessors.

## With TypeScript

If you want type safety for your class names, add a declaration file:

```typescript
// src/types/css.d.ts
declare module "*.module.css" {
  const classes: { readonly [key: string]: string };
  export default classes;
}
```

Now TypeScript will autocomplete your class names when you type `styles.`.

## Benefits and Trade-offs

**Benefits:**
- Zero runtime cost (transformed at build time)
- No class name conflicts ever
- Works with regular CSS (no new syntax to learn)
- Great for component-level styling

**Trade-offs:**
- No dynamic styles based on props (use inline styles alongside)
- No built-in theming (you need CSS variables)
- Verbose for small style tweaks

CSS Modules hit a sweet spot for many projects. They give you the power of real CSS without the global scope problems.
