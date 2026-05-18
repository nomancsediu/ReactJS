# Controlled Forms

Controlled forms are the "React way" of handling form inputs. Instead of letting the DOM manage the input values, React state becomes the single source of truth. I found this concept confusing at first, but it clicks once you see it in action.

## What Is a Controlled Input?

A controlled input has its value tied to React state, and its `onChange` handler updates that state:

```jsx
function SimpleForm() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

Every time the user types, the state updates, and React re-renders the input with the new value. React is in full control of what the input displays.

## Handling Multiple Fields

When I first built forms, I created separate state for each field. That works for two or three inputs, but it gets messy quickly:

```jsx
// This approach does not scale well
const [name, setName] = useState("");
const [email, setEmail] = useState("");
const [age, setAge] = useState("");
```

A better approach is using a single state object:

```jsx
function RegistrationForm() {
  const [form, setForm] = useState({
    name: "",
    email: "",
    age: "",
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm((prev) => ({
      ...prev,
      [name]: value,
    }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log(form);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" value={form.name} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
      <input name="age" value={form.age} onChange={handleChange} />
      <button type="submit">Register</button>
    </form>
  );
}
```

Notice how a single `handleChange` function works for all inputs. The `name` attribute on each input matches the key in the state object. This pattern scales well.

## Dynamic Styles and Validation

Because the state is always current, you can easily add dynamic behavior:

```jsx
<input
  name="email"
  value={form.email}
  onChange={handleChange}
  style={{ borderColor: form.email && !form.email.includes("@") ? "red" : "" }}
/>
```

## The Trade-off

Controlled forms give you full control, but they re-render on every keystroke. For most forms this is fine. For very large forms or performance-sensitive scenarios, libraries like React Hook Form can help by reducing re-renders. We will cover that next.
