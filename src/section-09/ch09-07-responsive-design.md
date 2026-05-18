# Responsive Design

Your React app will be viewed on phones, tablets, laptops, and large desktops. Making it look good on all of them is not optional; it is essential. Let me share the patterns and techniques I use for responsive design.

## Mobile-First Approach

The mobile-first philosophy is simple: write styles for the smallest screen first, then add complexity for larger screens. This works naturally with how CSS cascades. Base styles apply everywhere, and media queries add or override styles at larger breakpoints.

```css
/* Mobile styles (default) */
.container {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 24px;
    flex-direction: row;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    padding: 32px;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

With Tailwind, this becomes even cleaner:

```jsx
<div className="
  flex flex-col gap-4 p-4
  md:flex-row md:p-6
  lg:max-w-6xl lg:mx-auto lg:p-8
">
  {/* content */}
</div>
```

## Media Queries

Media queries are the foundation of responsive design. The most common ones:

```css
/* Viewport width */
@media (min-width: 768px) { /* styles for screens 768px and wider */ }
@media (max-width: 767px) { /* styles for screens narrower than 768px */ }

/* Orientation */
@media (orientation: landscape) { /* styles for landscape mode */ }

/* Prefers color scheme */
@media (prefers-color-scheme: dark) { /* dark mode styles */ }

/* Prefers reduced motion */
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; }
}
```

I always prefer `min-width` queries (mobile-first) over `max-width` (desktop-first). It produces less CSS and follows a more logical progression.

## Container Queries

Container queries are a newer feature that lets components respond to their container size rather than the viewport. This is a game-changer for reusable components:

```css
.card-wrapper {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: flex;
    gap: 16px;
  }
}

@container (max-width: 399px) {
  .card {
    display: block;
  }
}
```

Now a card component adapts based on how much space its parent gives it, not the screen size. This makes truly reusable components possible.

## Common Responsive Patterns

### Navigation

```jsx
function Navbar() {
  const [menuOpen, setMenuOpen] = useState(false);

  return (
    <nav className="flex items-center justify-between p-4">
      <span className="font-bold text-xl">MyApp</span>

      {/* Desktop menu */}
      <div className="hidden md:flex gap-6">
        <a href="/home">Home</a>
        <a href="/about">About</a>
        <a href="/contact">Contact</a>
      </div>

      {/* Mobile hamburger */}
      <button
        className="md:hidden"
        onClick={() => setMenuOpen(!menuOpen)}
      >
        Menu
      </button>

      {/* Mobile dropdown */}
      {menuOpen && (
        <div className="md:hidden absolute top-16 left-0 right-0 bg-white p-4 shadow-lg">
          <a href="/home" className="block py-2">Home</a>
          <a href="/about" className="block py-2">About</a>
          <a href="/contact" className="block py-2">Contact</a>
        </div>
      )}
    </nav>
  );
}
```

### Grid Layouts

```jsx
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">
  {products.map((product) => (
    <ProductCard key={product.id} product={product} />
  ))}
</div>
```

Responsive design is not a feature you add at the end. Build it in from the start, and your app will work beautifully everywhere.
