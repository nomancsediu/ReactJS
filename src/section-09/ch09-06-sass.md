# SASS in React

SASS (Syntactically Awesome Style Sheets) adds features that regular CSS lacks: variables, nesting, mixins, and functions. It has been around for years and remains a solid choice for React projects that need more CSS power without going the CSS-in-JS route.

## SASS vs SCSS

SASS has two syntaxes. SASS uses indentation (no braces, no semicolons). SCSS uses braces and semicolons, making it a superset of regular CSS. I always use SCSS because it is easier to read and integrates smoothly with existing CSS.

## Installing SASS

```bash
npm install sass
```

Vite supports SASS out of the box. Just create `.scss` or `.module.scss` files and import them.

## Variables

Variables store reusable values like colors, sizes, and fonts:

```scss
// _variables.scss
$primary-color: #16a34a;
$secondary-color: #e5e7eb;
$text-color: #1f2937;
$border-radius: 8px;
$font-stack: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
```

```scss
// Button.module.scss
@use "../variables" as *;

.button {
  padding: 10px 20px;
  border-radius: $border-radius;
  font-family: $font-stack;
  cursor: pointer;

  &.primary {
    background-color: $primary-color;
    color: white;
  }

  &.secondary {
    background-color: $secondary-color;
    color: $text-color;
  }
}
```

## Nesting

Nesting lets you write styles that follow the HTML structure:

```scss
.card {
  padding: 16px;
  border-radius: 8px;
  background: white;

  &-title {
    font-size: 18px;
    font-weight: bold;
  }

  &-body {
    margin-top: 8px;
    color: #6b7280;
  }

  &:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }
}
```

The `&` refers to the parent selector. `&-title` becomes `.card-title`, and `&:hover` becomes `.card:hover`.

## Mixins

Mixins are reusable style blocks that can accept arguments:

```scss
// _mixins.scss
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin responsive($breakpoint) {
  @if $breakpoint == "sm" {
    @media (min-width: 640px) { @content; }
  } @else if $breakpoint == "md" {
    @media (min-width: 768px) { @content; }
  } @else if $breakpoint == "lg" {
    @media (min-width: 1024px) { @content; }
  }
}
```

```scss
@use "../mixins" as *;

.spinner {
  @include flex-center;
  width: 40px;
  height: 40px;

  @include responsive("md") {
    width: 60px;
    height: 60px;
  }
}
```

## Using SCSS Modules in React

```tsx
import styles from "./Card.module.scss";

function Card({ title, children }) {
  return (
    <div className={styles.card}>
      <h2 className={styles.cardTitle}>{title}</h2>
      <div className={styles.cardBody}>{children}</div>
    </div>
  );
}
```

SASS with CSS Modules gives you the best of both worlds: the preprocessor features of SASS with the scoped styles of modules. It is a mature, reliable combination.
