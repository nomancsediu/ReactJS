# Component Testing

Now we get to the fun part: testing real React components. This is where everything comes together. Rendering, finding elements, simulating interactions, and making assertions. Let me walk through the patterns I use every day.

## Rendering and Finding Elements

The basic test renders a component and checks that the right content appears:

```jsx
import { render, screen } from "@testing-library/react";
import UserProfile from "./UserProfile";

test("displays user information", () => {
  render(<UserProfile name="Alice" role="Admin" />);

  expect(screen.getByText("Alice")).toBeInTheDocument();
  expect(screen.getByText("Admin")).toBeInTheDocument();
});
```

Use `getByRole` when you can. It tests accessibility at the same time:

```jsx
test("renders a heading", () => {
  render(<PageTitle title="Dashboard" />);

  expect(screen.getByRole("heading", { name: /dashboard/i })).toBeInTheDocument();
});
```

## Simulating User Events

`userEvent` simulates real user interactions better than `fireEvent`. It fires all the events a real user would trigger:

```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import Counter from "./Counter";

test("increments counter on click", async () => {
  const user = userEvent.setup();
  render(<Counter />);

  expect(screen.getByText("Count: 0")).toBeInTheDocument();

  await user.click(screen.getByRole("button", { name: /increment/i }));

  expect(screen.getByText("Count: 1")).toBeInTheDocument();
});
```

Always `await` user events. They are asynchronous and need time to process.

## Testing Forms

Forms are one of the most common things to test:

```jsx
test("submits form with user input", async () => {
  const handleSubmit = jest.fn();
  const user = userEvent.setup();

  render(<LoginForm onSubmit={handleSubmit} />);

  await user.type(screen.getByLabelText("Email"), "alice@test.com");
  await user.type(screen.getByLabelText("Password"), "secret123");
  await user.click(screen.getByRole("button", { name: /log in/i }));

  expect(handleSubmit).toHaveBeenCalledWith({
    email: "alice@test.com",
    password: "secret123",
  });
});
```

## Testing Conditional Rendering

Check that the right content appears or disappears based on state:

```jsx
test("shows error message when login fails", async () => {
  const user = userEvent.setup();
  render(<LoginForm />);

  // Error is not shown initially
  expect(screen.queryByText("Invalid credentials")).not.toBeInTheDocument();

  // Trigger a failed login
  await user.click(screen.getByRole("button", { name: /log in/i }));

  // Now the error appears
  expect(screen.getByText("Invalid credentials")).toBeInTheDocument();
});
```

Use `queryBy` for checking that something is NOT present. `getBy` throws an error if the element is missing, which is the wrong behavior for a negative assertion.

## Testing Loading States

Async components show loading states that transition to content:

```jsx
test("shows loading then data", async () => {
  render(<UserList />);

  // Loading state
  expect(screen.getByText("Loading...")).toBeInTheDocument();

  // Wait for data to appear
  const userItem = await screen.findByText("Alice");
  expect(userItem).toBeInTheDocument();

  // Loading is gone
  expect(screen.queryByText("Loading...")).not.toBeInTheDocument();
});
```

`findBy` waits for the element to appear. It uses `waitFor` under the hood with a default timeout of 1000ms.

## Assertions with jest-dom

RTL extends Jest with DOM-specific matchers:

```jsx
expect(element).toBeInTheDocument();
expect(element).toBeVisible();
expect(element).toBeDisabled();
expect(element).toHaveTextContent("Hello");
expect(element).toHaveClass("active");
expect(element).toHaveAttribute("href", "/about");
expect(element).toHaveFocus();
```

These matchers make your tests read like English. "Expect the button to be disabled." That clarity is worth a lot when you come back to tests months later.

These patterns cover most component testing scenarios. Render, interact, assert. That is the cycle.
