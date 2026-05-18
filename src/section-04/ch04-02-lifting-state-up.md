# Lifting State Up

Here is a problem I ran into early on: two sibling components needed the same data. Component A had the state, but Component B needed it too. How do you share state between siblings?

The answer is lifting state up. You move the state to their closest common parent, then pass it down as props.

## The Problem

Imagine a temperature converter. You have a Celsius input and a Fahrenheit input. When you change one, the other should update too.

```jsx
// This will NOT work
function CelsiusInput() {
  const [celsius, setCelsius] = useState(0);
  return <input value={celsius} onChange={e => setCelsius(e.target.value)} />;
}

function FahrenheitInput() {
  const [fahrenheit, setFahrenheit] = useState(32);
  return <input value={fahrenheit} onChange={e => setFahrenheit(e.target.value)} />;
}
```

Each input has its own state. They cannot talk to each other.

## The Solution

Lift the state to the parent and pass it down:

```jsx
function TemperatureConverter() {
  const [celsius, setCelsius] = useState(0);

  const fahrenheit = (celsius * 9) / 5 + 32;

  const handleCelsiusChange = (value) => {
    setCelsius(parseFloat(value) || 0);
  };

  const handleFahrenheitChange = (value) => {
    const f = parseFloat(value) || 0;
    setCelsius(((f - 32) * 5) / 9);
  };

  return (
    <div>
      <CelsiusInput value={celsius} onChange={handleCelsiusChange} />
      <FahrenheitInput value={fahrenheit} onChange={handleFahrenheitChange} />
    </div>
  );
}

function CelsiusInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}

function FahrenheitInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={(e) => onChange(e.target.value)}
    />
  );
}
```

Now the parent owns the single source of truth. Both inputs are controlled by the parent, and they stay in sync.

## The Common Parent Pattern

The rule is simple: lift state to the lowest common ancestor that needs to access or modify it. Not higher than necessary.

If two siblings need shared state, lift to their parent. If two cousins need shared state, lift to their grandparent. Keep it as close to where it is used as possible.

## A Practical Example

I use this pattern all the time for things like:

- A search input and a filtered list that both need the query
- A shopping cart counter in the header and an "Add to Cart" button in a product card
- A theme toggle and components that read the theme

```jsx
function ProductPage() {
  const [cartCount, setCartCount] = useState(0);

  return (
    <div>
      <Header cartCount={cartCount} />
      <ProductCard onAddToCart={() => setCartCount(c => c + 1)} />
    </div>
  );
}
```

Lifting state is probably the pattern I use most. Whenever two components need the same data, lift it up.
