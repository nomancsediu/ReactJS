# React Hook Form

After building a few controlled forms, I quickly realized the re-render problem. Every keystroke triggers a state update and a re-render. For small forms it is fine, but for bigger ones it slows things down. That is where React Hook Form (RHF) comes in.

## Why React Hook Form?

RHF uses uncontrolled inputs under the hood. It tracks form values via refs instead of state, which means fewer re-renders. It also handles validation, error messages, and form submission with very little code.

## Basic Setup

```bash
npm install react-hook-form
```

```jsx
import { useForm } from "react-hook-form";

function SignUpForm() {
  const { register, handleSubmit } = useForm();

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name")} placeholder="Name" />
      <input {...register("email")} placeholder="Email" />
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

That `register` function connects each input to the form. The spread operator passes the `name`, `onChange`, `onBlur`, and `ref` props automatically. It feels like magic the first time you see it.

## Built-in Validation Rules

RHF has validation rules you can pass directly to `register`:

```jsx
<input
  {...register("email", {
    required: "Email is required",
    pattern: {
      value: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      message: "Invalid email address",
    },
  })}
/>
```

```jsx
<input
  {...register("password", {
    required: "Password is required",
    minLength: {
      value: 8,
      message: "Password must be at least 8 characters",
    },
  })}
/>
```

## Displaying Errors

RHF provides an `errors` object you can use to show validation messages:

```jsx
const { register, handleSubmit, formState: { errors } } = useForm();

return (
  <form onSubmit={handleSubmit(onSubmit)}>
    <input {...register("name", { required: "Name is required" })} />
    {errors.name && <p className="text-red-500">{errors.name.message}</p>}
    
    <button type="submit">Submit</button>
  </form>
);
```

## Performance Benefits

The big win with RHF is performance. Because it uses refs instead of state, typing in one field does not cause the entire component to re-render. I noticed the difference immediately in a form with 20+ fields. The input felt snappy compared to the controlled approach.

RHF also isolates re-renders to only the fields that have errors, which makes error display efficient too. In the next chapter, we will add Zod for even more powerful validation.
