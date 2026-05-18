# Complex State with useReducer

When `useState` gets messy with too many related pieces of state, `useReducer` brings order. I avoided it for a long time because it looked like Redux in miniature, and Redux scared me. But `useReducer` is actually simple once you understand the pattern.

## The Problem with useState

Imagine managing a form with multiple fields and validation:

```jsx
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitted, setSubmitted] = useState(false);

  // Updating is scattered and error-prone
  const handleSubmit = () => {
    setIsSubmitting(true);
    const newErrors = {};
    if (!name) newErrors.name = 'Required';
    if (!email) newErrors.email = 'Required';
    setErrors(newErrors);
    // ... more scattered updates
  };
}
```

Each piece of state is independent, but they are really part of one logical unit. Updating them separately can lead to inconsistent intermediate states.

## Enter useReducer

`useReducer` groups related state into a single object and handles updates through dispatched actions:

```jsx
const initialState = {
  name: '',
  email: '',
  errors: {},
  isSubmitting: false,
  submitted: false,
};

function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return { ...state, [action.field]: action.value };
    case 'SET_ERRORS':
      return { ...state, errors: action.errors };
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true };
    case 'SUBMIT_SUCCESS':
      return { ...state, isSubmitting: false, submitted: true };
    case 'RESET':
      return initialState;
    default:
      return state;
  }
}
```

```jsx
function Form() {
  const [state, dispatch] = useReducer(formReducer, initialState);

  const handleSubmit = async () => {
    const errors = {};
    if (!state.name) errors.name = 'Required';
    if (!state.email) errors.email = 'Required';

    if (Object.keys(errors).length > 0) {
      dispatch({ type: 'SET_ERRORS', errors });
      return;
    }

    dispatch({ type: 'SUBMIT_START' });
    await submitForm(state);
    dispatch({ type: 'SUBMIT_SUCCESS' });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={state.name}
        onChange={(e) => dispatch({ type: 'SET_FIELD', field: 'name', value: e.target.value })}
      />
      {state.errors.name && <span>{state.errors.name}</span>}
      <input
        value={state.email}
        onChange={(e) => dispatch({ type: 'SET_FIELD', field: 'email', value: e.target.value })}
      />
      {state.errors.email && <span>{state.errors.email}</span>}
      <button type="submit" disabled={state.isSubmitting}>
        {state.isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Action Types

Actions are objects with a `type` property and optional payload. I like to keep them simple and descriptive:

```jsx
// Good: clear and specific
dispatch({ type: 'ADD_ITEM', item: newItem });
dispatch({ type: 'REMOVE_ITEM', id: 3 });
dispatch({ type: 'CLEAR_CART' });

// Avoid: too generic
dispatch({ type: 'UPDATE', payload: anything });
```

Naming action types as uppercase verbs is a convention, not a requirement, but it makes them easy to spot in code.

## Dispatch Patterns

The `dispatch` function is stable across renders, which means you can safely pass it to child components or use it in effects without adding it to dependency arrays:

```jsx
// dispatch is stable, no need to add to deps
useEffect(() => {
  dispatch({ type: 'INITIALIZE', data: savedData });
}, []);
```

## When to Use useReducer

I reach for `useReducer` when:

- State has multiple related fields
- The next state depends on the previous state in complex ways
- I want a clear log of all state transitions
- Multiple components need to trigger the same state changes

For simple state like a toggle or a counter, `useState` is still the right choice. Do not overcomplicate things.
