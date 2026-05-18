# Inline Styles

Inline styles are the simplest way to style React components. You pass a JavaScript object to the `style` prop, and React applies those styles directly to the element. No CSS files needed.

## The Style Prop

```jsx
function WelcomeBanner() {
  return (
    <div style={{
      backgroundColor: "#f0f9ff",
      padding: "20px",
      borderRadius: "8px",
    }}>
      <h1 style={{ color: "#0369a1", fontSize: "24px" }}>
        Welcome Back!
      </h1>
    </div>
  );
}
```

Notice that CSS properties use camelCase instead of kebab-case. `background-color` becomes `backgroundColor`, `font-size` becomes `fontSize`, and `border-radius` becomes `borderRadius`.

## Dynamic Styles

Inline styles shine when you need dynamic styling based on state or props:

```jsx
function ProgressBar({ percent }) {
  return (
    <div style={{
      width: "100%",
      height: "8px",
      backgroundColor: "#e5e7eb",
      borderRadius: "4px",
    }}>
      <div style={{
        width: `${percent}%`,
        height: "100%",
        backgroundColor: percent > 80 ? "#22c55e" : "#3b82f6",
        borderRadius: "4px",
        transition: "width 0.3s ease",
      }} />
    </div>
  );
}
```

Changing styles based on data is easy because it is just JavaScript.

## Conditional Styles

You can merge style objects conditionally:

```jsx
function Alert({ type }) {
  const baseStyles = {
    padding: "12px 16px",
    borderRadius: "6px",
    fontSize: "14px",
  };

  const typeStyles = {
    success: { backgroundColor: "#dcfce7", color: "#166534" },
    error: { backgroundColor: "#fee2e2", color: "#991b1b" },
    warning: { backgroundColor: "#fef3c7", color: "#92400e" },
  };

  return (
    <div style={{ ...baseStyles, ...typeStyles[type] }}>
      Alert message
    </div>
  );
}
```

## Limitations

Inline styles have real drawbacks that you should know about:

**No pseudo-selectors.** You cannot style `:hover`, `:focus`, or `:active` with inline styles. This is a big limitation for interactive elements.

**No media queries.** Responsive design requires CSS files or a library.

**No animations.** Keyframe animations need CSS files or a library like Framer Motion.

**String values everywhere.** Numeric values work for some properties (`width: 200` becomes `200px`), but most need explicit units as strings.

**No cascading.** Each element gets its own style object. You cannot write rules that affect child elements.

```jsx
// This does NOT work
<div style={{ "h1.fontSize": "24px" }}>
  <h1>Title</h1>
</div>
```

Inline styles are great for quick, dynamic tweaks. For anything more complex, use CSS Modules, Tailwind, or another approach. We will cover those next.
