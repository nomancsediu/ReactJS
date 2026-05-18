# Tailwind Components

Now that Tailwind is set up, let us build real UI components. Tailwind's utility classes cover almost everything you need. I will show you the patterns I use most often.

## Common Utility Patterns

### Spacing and Sizing

```jsx
// Padding and margin
<div className="p-4">          /* padding: 16px */
<div className="px-6 py-3">    /* padding: 12px 24px */
<div className="mt-4 mb-2">    /* margin: 16px top, 8px bottom */

// Width and height
<div className="w-full">       /* width: 100% */
<div className="max-w-md">     /* max-width: 28rem */
<div className="h-screen">     /* height: 100vh */
```

### Typography

```jsx
<h1 className="text-3xl font-bold text-gray-900">Heading</h1>
<p className="text-base text-gray-600 leading-relaxed">Paragraph text</p>
<span className="text-sm font-medium text-gray-500">Small label</span>
```

### Flexbox and Grid

```jsx
// Flexbox
<div className="flex items-center justify-between gap-4">
  <span>Left</span>
  <span>Right</span>
</div>

// Grid
<div className="grid grid-cols-3 gap-4">
  <div>Item 1</div>
  <div>Item 2</div>
  <div>Item 3</div>
</div>
```

## Responsive Design

Tailwind uses mobile-first breakpoints. Styles without a prefix apply to all screen sizes. Prefixes like `sm:`, `md:`, and `lg:` add styles at larger breakpoints:

```jsx
<div className="
  grid
  grid-cols-1        /* 1 column on mobile */
  sm:grid-cols-2     /* 2 columns on small screens */
  lg:grid-cols-3     /* 3 columns on large screens */
  gap-6
  p-4
  md:p-8             /* more padding on larger screens */
">
  {/* cards */}
</div>
```

The default breakpoints are:
- `sm:` 640px
- `md:` 768px
- `lg:` 1024px
- `xl:` 1280px
- `2xl:` 1536px

## Hover, Focus, and Active States

Tailwind makes interactive states easy:

```jsx
<button className="
  bg-green-600
  text-white
  px-6
  py-2
  rounded-lg
  hover:bg-green-700       /* darker on hover */
  focus:outline-none
  focus:ring-2
  focus:ring-green-500     /* ring on focus */
  focus:ring-offset-2
  active:bg-green-800      /* even darker when pressed */
  transition-colors         /* smooth transition */
  disabled:opacity-50       /* faded when disabled */
  disabled:cursor-not-allowed
">
  Save Changes
</button>
```

## Building a Card Component

Here is a complete card built entirely with Tailwind:

```jsx
function ProductCard({ name, price, image, onAddToCart }) {
  return (
    <div className="bg-white rounded-xl shadow-md overflow-hidden hover:shadow-lg transition-shadow">
      <img src={image} alt={name} className="w-full h-48 object-cover" />
      <div className="p-4">
        <h3 className="text-lg font-semibold text-gray-800">{name}</h3>
        <p className="text-green-600 font-bold mt-1">${price}</p>
        <button
          onClick={onAddToCart}
          className="mt-4 w-full bg-green-600 text-white py-2 rounded-lg hover:bg-green-700 transition-colors"
        >
          Add to Cart
        </button>
      </div>
    </div>
  );
}
```

## Custom Utilities

When you need something Tailwind does not provide, use arbitrary values with square brackets:

```jsx
<div className="top-[120px]">              /* custom position */
<div className="grid-cols-[1fr_2fr_1fr]">  /* custom grid */
<p className="text-[17px]">                /* custom font size */
```

Tailwind makes it fast to go from idea to styled component. The utility approach takes a few days to get comfortable with, but once it clicks, you will never want to go back.
