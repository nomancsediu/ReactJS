# React Testing Library

React Testing Library (RTL) changed how I think about testing. Before RTL, I used Enzyme and tested component internals: state values, instance methods, child component props. It was fragile. Every refactor broke my tests. RTL takes a completely different approach.

## The RTL Philosophy

RTL's guiding principle is: **test components the way users use them.** Users find elements by visible text, by role, by label. They do not find elements by CSS class or component name. Your tests should do the same.

```jsx
// Bad: finding by implementation detail
wrapper.find(".submit-button");

// Good: finding by accessible role
screen.getByRole("button", { name: /submit/i });
```

This approach makes your tests resilient to refactoring. Change your CSS, restructure your components, rename your variables. If the user experience stays the same, your tests pass.

## The Three Query Types

RTL gives you three families of queries:

**getBy*** - Finds one element. Throws an error if not found or if multiple found.

```jsx
const button = screen.getByRole("button", { name: /submit/i });
const heading = screen.getByText("Welcome");
const input = screen.getByLabelText("Email");
```

**queryBy*** - Finds one element. Returns `null` if not found. Use this to assert something is NOT present.

```jsx
expect(screen.queryByText("Error")).not.toBeInTheDocument();
```

**findBy*** - Finds one element asynchronously. Waits for it to appear. Returns a promise.

```jsx
const message = await screen.findByText("Saved successfully");
```

The plural versions (`getAllBy`, `queryAllBy`, `findAllBy`) return arrays instead of single elements.

## Priority Order for Queries

RTL recommends this priority for selecting elements:

1. **getByRole** - Best option. Tests accessibility at the same time.
2. **getByLabelText** - Great for form fields.
3. **getByPlaceholderText** - OK for inputs without labels.
4. **getByText** - Good for buttons, headings, and links.
5. **getByDisplayValue** - For checking current input values.
6. **getByTestId** - Last resort. Use only when no other query works.

```jsx
// Best: by role
screen.getByRole("button", { name: /save/i });

// Good: by text
screen.getByText("Save");

// Acceptable: by label
screen.getByLabelText("Save changes");

// Last resort: by test ID
screen.getByTestId("save-button");
```

## The Rendering Function

`render` is how you mount a component in a test:

```jsx
import { render, screen } from "@testing-library/react";
import UserCard from "./UserCard";

test("shows user name and email", () => {
  render(<UserCard name="Alice" email="alice@test.com" />);

  expect(screen.getByText("Alice")).toBeInTheDocument();
  expect(screen.getByText("alice@test.com")).toBeInTheDocument();
});
```

RTL automatically cleans up after each test. You do not need to unmount manually.

## Why This Matters

When you test by role and text, you are also testing accessibility. If a button has no accessible name, `getByRole` will not find it. That means your test is catching an accessibility bug at the same time it is catching a functional bug.

Testing like a user makes your tests more valuable and more maintainable. It took me a while to stop reaching for CSS selectors, but once I did, my tests became much more reliable.
