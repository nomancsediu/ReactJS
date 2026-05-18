# Controlled vs Uncontrolled Components

One of the first confusing things I hit in React was form inputs. Should I store the value in state or just let the DOM handle it? It turns out both approaches are valid, and each has its place.

## Controlled Components

A controlled component means React controls the input value through state. Every keystroke updates state, and the input reads from state.

```jsx
function ControlledForm() {
  const [name, setName] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(name);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Your name"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

The input value always reflects `name` state. If you want to modify the input programmatically, just update the state. This makes validation and conditional rendering easy because you always know the current value.

## Uncontrolled Components

An uncontrolled component lets the DOM manage the value. You access it with a ref when you need it.

```jsx
function UncontrolledForm() {
  const nameRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(nameRef.current.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={nameRef} defaultValue="Guest" placeholder="Your name" />
      <button type="submit">Submit</button>
    </form>
  );
}
```

Notice `defaultValue` instead of `value`. This sets the initial value but then React steps back and lets the DOM handle changes.

## When to Use Which

I use controlled components most of the time. They make it easy to:

- Validate input as the user types
- Disable buttons based on form state
- Format or transform input values
- Reset forms programmatically

I reach for uncontrolled components when:

- I am integrating with non-React code
- The form is simple and I just need the value on submit
- File inputs, which are always uncontrolled in React

## useRef for Forms

For a quick form that does not need real-time validation, refs are perfectly fine:

```jsx
function SimpleForm() {
  const emailRef = useRef(null);
  const passwordRef = useRef(null);

  const handleSubmit = (e) => {
    e.preventDefault();
    const data = {
      email: emailRef.current.value,
      password: passwordRef.current.value,
    };
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input ref={emailRef} type="email" />
      <input ref={passwordRef} type="password" />
      <button type="submit">Log In</button>
    </form>
  );
}
```

My rule of thumb: start with controlled. Switch to uncontrolled only when you have a reason.
