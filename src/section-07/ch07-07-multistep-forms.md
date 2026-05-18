# Multi-step Forms

Not every form fits on one screen. Sign-up flows, checkout processes, and onboarding wizards all benefit from breaking a long form into steps. But multi-step forms add complexity around state management and per-step validation. Let me walk you through how I build them.

## Step Management

The core idea is tracking which step the user is on and conditionally rendering the right fields:

```jsx
function MultiStepForm() {
  const [step, setStep] = useState(1);
  const totalSteps = 3;

  const next = () => setStep((s) => Math.min(s + 1, totalSteps));
  const prev = () => setStep((s) => Math.max(s - 1, 1));

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {step === 1 && <PersonalInfoStep />}
      {step === 2 && <AddressStep />}
      {step === 3 && <ReviewStep />}
      <div className="flex justify-between">
        {step > 1 && <button type="button" onClick={prev}>Back</button>}
        {step < totalSteps ? (
          <button type="button" onClick={next}>Next</button>
        ) : (
          <button type="submit">Submit</button>
        )}
      </div>
    </form>
  );
}
```

## Validation Per Step

This is the tricky part. You only want to validate the fields on the current step before allowing the user to proceed. With React Hook Form, you can use `trigger` to validate specific fields:

```jsx
function MultiStepForm() {
  const { register, handleSubmit, trigger, formState: { errors } } = useForm({
    resolver: zodResolver(fullSchema),
  });

  const next = async () => {
    const stepFields = {
      1: ["name", "email"],
      2: ["address", "city", "zip"],
      3: [], // review step, no new fields
    };

    const isValid = await trigger(stepFields[step]);
    if (isValid) {
      setStep((s) => s + 1);
    }
  };

  // ...
}
```

The `trigger` function validates only the fields you specify. If they all pass, the user moves to the next step.

## State Persistence

Because all steps share the same `useForm` instance, the state is automatically preserved when the user goes back. They will see their previously entered values still filled in. No extra work needed.

## Alternative: Separate Schemas Per Step

For larger forms, I prefer defining a schema for each step:

```jsx
const stepOneSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

const stepTwoSchema = z.object({
  address: z.string().min(5),
  city: z.string().min(2),
});

const fullSchema = stepOneSchema.merge(stepTwoSchema).extend({
  terms: z.boolean().refine((val) => val === true),
});
```

This way each step only validates its own fields, but the full schema ensures everything is valid at submission time.

Multi-step forms take a bit more planning, but the improved user experience is worth it. Nobody likes scrolling through an endless form.
