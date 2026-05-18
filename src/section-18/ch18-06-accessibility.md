# Accessibility in React

Accessibility (often shortened to a11y, because there are 11 letters between a and y) means making your app usable by everyone, including people using screen readers, keyboard navigation, or other assistive technology. I used to treat accessibility as an afterthought. Now I realize it is just part of building things right.

## Semantic HTML

The most impactful thing you can do is use the right HTML elements:

```tsx
// Bad: divs for everything
<div onClick={handleClick}>Click me</div>

// Good: use a button
<button onClick={handleClick}>Click me</button>
```

Buttons are focusable, respond to Enter and Space, and screen readers announce them as interactive. Divs are none of those things.

Other semantic elements:

```tsx
// Use headings in order
<h1>Main Title</h1>
<h2>Section</h2>
<h3>Subsection</h3>

// Use lists for lists
<ul>
  <li>Item 1</li>
  <li>Item 2</li>
</ul>

// Use nav for navigation
<nav>
  <a href="/">Home</a>
  <a href="/about">About</a>
</nav>

// Use main and aside
<main>{content}</main>
<aside>{sidebar}</aside>
```

## ARIA Attributes

When semantic HTML is not enough, ARIA attributes fill the gaps:

```tsx
// Custom button that acts like a real button
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === "Enter" || e.key === " ") handleClick();
  }}
  aria-label="Close dialog"
>
  X
</div>
```

Common ARIA attributes I use:

```tsx
// Label elements that have no visible text
<button aria-label="Close menu">&times;</button>

// Describe the current state
<button aria-expanded={isOpen}>Menu</button>

// Indicate loading state
<div aria-busy={loading} aria-live="polite">
  {loading ? "Loading..." : content}
</div>

// Required form fields
<input aria-required="true" />

// Link a label to an input
<label htmlFor="email">Email</label>
<input id="email" type="email" />
```

## Keyboard Navigation

Every interactive element should work with the keyboard:

```tsx
function Dropdown({ options }: { options: string[] }) {
  const [isOpen, setIsOpen] = useState(false);
  const [activeIndex, setActiveIndex] = useState(-1);

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case "ArrowDown":
        e.preventDefault();
        setActiveIndex((i) => Math.min(i + 1, options.length - 1));
        break;
      case "ArrowUp":
        e.preventDefault();
        setActiveIndex((i) => Math.max(i - 1, 0));
        break;
      case "Enter":
        if (activeIndex >= 0) {
          // Select the option
        }
        break;
      case "Escape":
        setIsOpen(false);
        break;
    }
  };

  return (
    <div onKeyDown={handleKeyDown}>
      <button onClick={() => setIsOpen(!isOpen)}>Select</button>
      {isOpen && (
        <ul role="listbox">
          {options.map((option, index) => (
            <li
              key={option}
              role="option"
              aria-selected={index === activeIndex}
            >
              {option}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## Focus Management

Move focus programmatically when needed, like after opening a modal:

```tsx
function Modal({ isOpen, onClose }: { isOpen: boolean; onClose: () => void }) {
  const closeRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) {
      closeRef.current?.focus();
    }
  }, [isOpen]);

  return isOpen ? (
    <div role="dialog" aria-modal="true">
      <button ref={closeRef} onClick={onClose}>Close</button>
    </div>
  ) : null;
}
```

## Quick Accessibility Checklist

- All images have `alt` text
- Forms have labels linked to inputs
- Interactive elements are keyboard accessible
- Color is not the only way to convey information
- Focus is visible and logical
- ARIA attributes are used where semantic HTML falls short

Accessibility is not a feature you add at the end. It is a habit you build from the start. The good news is that most of it is just using HTML correctly, which is easier than you might think.
