# Integration Testing

Unit tests check individual pieces. Integration tests check how those pieces work together. In React, that means testing components that interact with each other, share context, and navigate between routes. I find integration tests the most valuable because they catch bugs that unit tests miss.

## Testing Component Interactions

A parent and child component working together is a common integration point:

```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import TodoApp from "./TodoApp";

test("adding and completing a todo", async () => {
  const user = userEvent.setup();
  render(<TodoApp />);

  // Add a todo
  await user.type(screen.getByPlaceholderText("New todo"), "Buy milk");
  await user.click(screen.getByRole("button", { name: /add/i }));

  // It appears in the list
  expect(screen.getByText("Buy milk")).toBeInTheDocument();

  // Complete it
  await user.click(screen.getByRole("checkbox", { name: /buy milk/i }));

  // It shows as completed
  expect(screen.getByText("Buy milk")).toHaveClass("line-through");

  // Active count updates
  expect(screen.getByText("0 items left")).toBeInTheDocument();
});
```

This test exercises the input, the add button, the list rendering, the checkbox, and the counter. All working together, just like a real user would experience.

## Testing Context

When components use React Context, you need to provide the context in tests:

```jsx
import { ThemeProvider } from "./ThemeContext";
import ThemedCard from "./ThemedCard";

test("renders with dark theme", () => {
  render(
    <ThemeProvider initialTheme="dark">
      <ThemedCard title="Hello" />
    </ThemeProvider>
  );

  const card = screen.getByText("Hello").closest(".card");
  expect(card).toHaveClass("dark");
});
```

For frequently used contexts, create a test helper:

```jsx
// test-utils.jsx
function renderWithProviders(ui, { theme = "light", ...options } = {}) {
  function Wrapper({ children }) {
    return <ThemeProvider initialTheme={theme}>{children}</ThemeProvider>;
  }
  return render(ui, { wrapper: Wrapper, ...options });
}

// Usage
test("dark themed card", () => {
  renderWithProviders(<ThemedCard title="Hello" />, { theme: "dark" });
  expect(screen.getByText("Hello").closest(".card")).toHaveClass("dark");
});
```

## Testing Router Navigation

Components that use React Router need a router in tests:

```jsx
import { MemoryRouter } from "react-router-dom";
import App from "./App";

test("navigates to about page", async () => {
  const user = userEvent.setup();
  render(
    <MemoryRouter initialEntries={["/"]}>
      <App />
    </MemoryRouter>
  );

  // On home page
  expect(screen.getByText("Welcome Home")).toBeInTheDocument();

  // Click the about link
  await user.click(screen.getByRole("link", { name: /about/i }));

  // Now on about page
  expect(screen.getByText("About Us")).toBeInTheDocument();
});
```

`MemoryRouter` lets you control the initial route and test navigation without a real browser URL bar.

## Testing Async Flows

Real apps have multi-step async flows. Test the complete sequence:

```jsx
test("user can create an account and see welcome message", async () => {
  const user = userEvent.setup();
  render(
    <MemoryRouter>
      <App />
    </MemoryRouter>
  );

  // Fill out signup form
  await user.type(screen.getByLabelText("Name"), "Alice");
  await user.type(screen.getByLabelText("Email"), "alice@test.com");
  await user.type(screen.getByLabelText("Password"), "secret123");
  await user.click(screen.getByRole("button", { name: /sign up/i }));

  // Wait for redirect and welcome message
  expect(await screen.findByText("Welcome, Alice!")).toBeInTheDocument();
  expect(screen.getByText("alice@test.com")).toBeInTheDocument();
});
```

Integration tests are your safety net. They verify that all your well-tested individual pieces actually work when combined. Write them for your most important user flows: sign up, checkout, data creation, anything where multiple components must cooperate.
