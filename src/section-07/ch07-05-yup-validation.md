# Yup Validation

Before I discovered Zod, I used Yup for schema validation. Yup has been around longer and is very popular in the React ecosystem. It works great with React Hook Form too. Let me show you how.

## What Is Yup?

Yup is a JavaScript schema builder for value parsing and validation. It lets you define object schemas with validation rules, similar to Zod. The main difference is that Yup was not built with TypeScript as a first-class concern.

## Installing Yup and the Resolver

```bash
npm install yup @hookform/resolvers
```

## Defining a Schema

Yup uses a chainable API with the `shape` method:

```jsx
import * as yup from "yup";

const loginSchema = yup.object().shape({
  email: yup
    .string()
    .required("Email is required")
    .email("Invalid email address"),
  password: yup
    .string()
    .required("Password is required")
    .min(8, "Password must be at least 8 characters"),
});
```

The `shape` method defines the structure of your object and its validation rules. Each field gets its own chain of validators.

## Cross-field Validation

You can reference other fields in Yup using `ref` and `oneOf`:

```jsx
const signUpSchema = yup.object().shape({
  password: yup.string().required("Password is required").min(8),
  confirmPassword: yup
    .string()
    .oneOf([yup.ref("password")], "Passwords must match")
    .required("Please confirm your password"),
});
```

## Integrating with React Hook Form

```tsx
import { useForm } from "react-hook-form";
import { yupResolver } from "@hookform/resolvers/yup";

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: yupResolver(loginSchema),
  });

  const onSubmit = (data) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("email")} />
      {errors.email && <p>{errors.email.message}</p>}

      <input type="password" {...register("password")} />
      {errors.password && <p>{errors.password.message}</p>}

      <button type="submit">Log In</button>
    </form>
  );
}
```

## Yup vs Zod

Here is my honest comparison:

| Feature | Yup | Zod |
|---------|-----|-----|
| TypeScript inference | Manual or extra setup | Built-in |
| Bundle size | Larger | Smaller |
| Community | Older, more examples | Growing fast |
| API style | Chainable methods | Chainable methods |
| Async validation | Supported | Supported |

I switched to Zod because the TypeScript integration is seamless. But if your project already uses Yup, there is no urgent reason to switch. Both are solid choices. Pick whichever feels more natural to you and your team.
