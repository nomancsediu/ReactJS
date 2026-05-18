# Form Errors

Validation is only useful if the user can see what went wrong. Showing clear, helpful error messages is one of the most important parts of form UX. I have used plenty of forms that just say "Invalid input" without telling me which field or why. Let us make sure our forms are not like that.

## Field-Level Errors

The most common approach is showing an error message right below or next to the field that has the problem. With React Hook Form, this is straightforward:

```jsx
const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: zodResolver(schema),
});

return (
  <form onSubmit={handleSubmit(onSubmit)}>
    <div>
      <label htmlFor="email">Email</label>
      <input id="email" {...register("email")} />
      {errors.email && (
        <span className="text-red-500 text-sm">{errors.email.message}</span>
      )}
    </div>
  </form>
);
```

Each field gets its own error check and its own message. This makes it easy for the user to spot exactly what needs fixing.

## Form-Level Errors

Sometimes an error is not tied to a specific field. Maybe the server rejected the submission, or the credentials are wrong. For these cases, you want a form-level error:

```jsx
const [serverError, setServerError] = useState("");

const onSubmit = async (data) => {
  try {
    setServerError("");
    await loginUser(data);
  } catch (error) {
    setServerError("Invalid email or password. Please try again.");
  }
};

return (
  <form onSubmit={handleSubmit(onSubmit)}>
    {serverError && (
      <div className="bg-red-50 border border-red-300 text-red-700 p-3 rounded mb-4">
        {serverError}
      </div>
    )}
    {/* fields */}
  </form>
);
```

## Styling Errors Consistently

I like to create a reusable error display component:

```jsx
function FieldError({ error }) {
  if (!error) return null;
  return (
    <p className="text-red-500 text-sm mt-1">{error.message}</p>
  );
}

// Usage
<input {...register("name", { required: "Name is required" })} />
<FieldError error={errors.name} />
```

## Error Timing

One thing I learned: showing errors too early is annoying. RHF validates on submit by default, which is usually the best experience. You can change this with the `mode` option:

```jsx
useForm({
  resolver: zodResolver(schema),
  mode: "onBlur",  // validate when field loses focus
  // mode: "onChange",  // validate on every keystroke
  // mode: "onTouched",  // validate after first touch, then on change
});
```

I prefer `onBlur` for most forms. It gives the user a chance to finish typing before showing errors, but still catches issues early.
