# Understanding Side Effects

Before we write any API calls, we need to understand what a side effect actually is. This concept confused me at first, but once it clicks, everything about `useEffect` makes sense.

## Pure Functions

A pure function always returns the same output for the same input. No surprises.

```js
// Pure: same input, same output, every time
function add(a, b) {
  return a + b;
}

// Not pure: result depends on something outside
let taxRate = 0.08;
function calculateTotal(price) {
  return price + price * taxRate; // depends on external variable
}
```

React components are supposed to be pure functions. You give them props and state, they return UI. That predictability is what makes React work. But real apps need to do impure things: fetch data, set timers, subscribe to events, manipulate the DOM directly.

Those impure things are **side effects**.

## What Counts as a Side Effect?

A side effect is anything that reaches outside the component's render cycle:

```jsx
function Example() {
  // These are all side effects:
  // - Fetching data from an API
  // - Setting up a subscription
  // - Manually changing the DOM
  // - Setting a timer
  // - Reading from localStorage

  // This is NOT a side effect:
  const fullName = `${firstName} ${lastName}`; // just computation
  return <h1>{fullName}</h1>;
}
```

## The Effect Lifecycle

`useEffect` lets you run side effects after React finishes rendering. Think of it as "after paint" code.

```jsx
useEffect(() => {
  // 1. Setup: runs after every render (by default)
  const subscription = subscribeToChat();

  // 2. Cleanup: runs before the next effect or on unmount
  return () => {
    subscription.unsubscribe();
  };
}, [dependency]); // 3. Dependency array: controls when the effect runs
```

The dependency array is the key:

```jsx
// Runs after every render
useEffect(() => { console.log("every render"); });

// Runs once after mount
useEffect(() => { console.log("once"); }, []);

// Runs when userId changes
useEffect(() => { fetchUser(userId); }, [userId]);
```

## Why Not Put Effects in the Render?

I tried this. It does not work well:

```jsx
function BadExample() {
  // This runs on EVERY render
  fetch("/api/data").then(res => res.json()).then(setData);
  // Your app will make hundreds of requests!
  return <div>{data}</div>;
}
```

Effects belong in `useEffect` because React needs to control when they run. Render functions should be pure. Effects handle the messy stuff separately.

That separation is what keeps your app predictable and fast. Once you internalize this split, writing effects becomes second nature.
