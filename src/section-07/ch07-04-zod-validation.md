# Zod Validation

React Hook Form has built-in validation rules, and they work for simple cases. But once your validation logic gets complex, inline rules become hard to manage. That is where Zod shines. Zod lets you define validation schemas as separate objects, and they integrate beautifully with RHF.

## What Is Zod?

Zod is a TypeScript-first schema validation library. You describe what your data should look like, and Zod validates it. The best part? Zod infers TypeScript types from your schemas, so your validation and your types always stay in sync.

## Installing Zod and the Resolver

```bash
npm install zod @hookform/resolvers
```

## Defining a Schema

```jsx
import { z } from "zod";

const signUpSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email address"),
  password: z.string().min(8, "Password must be at least 8 characters"),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Passwords do not match",
  path: ["confirmPassword"],
});
```

The `refine` method lets you add custom validation logic that involves multiple fields. Here I am checking that the password and confirm password match.

## Inferring Types

This is where Zod really wins for me:

```tsx
type SignUpData = z.infer<typeof signUpSchema>;
```

One line and you have a fully typed interface that matches your schema. No more keeping a separate interface and a validation schema in sync manually.

## Integrating with React Hook Form

Use the `zodResolver` to connect Zod to RHF:

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";

function SignUpForm() {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(signUpSchema),
  });

  const onSubmit = (data: SignUpData) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register("name")} />
      {errors.name && <p>{errors.name.message}</p>}

      <input {...register("email")} />
      {errors.email && <p>{errors.email.message}</p>}

      <input type="password" {...register("password")} />
      {errors.password && <p>{errors.password.message}</p>}

      <input type="password" {...register("confirmPassword")} />
      {errors.confirmPassword && <p>{errors.confirmPassword.message}</p>}

      <button type="submit">Sign Up</button>
    </form>
  );
}
```

## Custom Validations

Zod supports transforms, unions, intersections, and much more. Here is a custom transform example:

```jsx
const phoneSchema = z.string()
  .regex(/^\d{10}$/, "Must be 10 digits")
  .transform((val) => `(${val.slice(0,3)}) ${val.slice(3,6)}-${val.slice(6)}`);
```

Zod has become my go-to validation library because of the TypeScript integration alone. It saves so much time and prevents so many bugs.
