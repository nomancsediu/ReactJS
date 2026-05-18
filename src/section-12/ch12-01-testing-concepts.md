# Testing Concepts

Before we write any tests, let us understand the landscape. There are different kinds of tests, and each serves a different purpose. I was confused about this at first, so let me break it down simply.

## Unit Tests

A unit test checks one small piece of code in isolation. A function, a hook, a single component. No external dependencies, no network calls, no database.

```jsx
// Testing a pure function is the simplest unit test
function formatCurrency(amount) {
  return `$${amount.toFixed(2)}`;
}

test("formats currency correctly", () => {
  expect(formatCurrency(10)).toBe("$10.00");
  expect(formatCurrency(10.5)).toBe("$10.50");
  expect(formatCurrency(0)).toBe("$0.00");
});
```

Unit tests are fast. You can run thousands in seconds. They give you precise feedback about exactly what broke.

## Integration Tests

Integration tests check that multiple pieces work together. A component that uses a hook, renders child components, and responds to user input:

```jsx
test("adding an item updates the list", async () => {
  render(<ShoppingList />);

  await userEvent.type(screen.getByPlaceholderText("Item name"), "Milk");
  await userEvent.click(screen.getByText("Add"));

  expect(screen.getByText("Milk")).toBeInTheDocument();
});
```

This test checks the input, the button, and the list rendering all working together. It does not test each piece in isolation. It tests the behavior.

## End-to-End Tests

E2E tests simulate real user behavior in a real browser. They click through your actual app, often with a tool like Playwright or Cypress:

```js
// Using Playwright
test("user can log in and see dashboard", async ({ page }) => {
  await page.goto("http://localhost:3000/login");
  await page.fill("[name=email]", "user@example.com");
  await page.fill("[name=password]", "password123");
  await page.click("button[type=submit]");
  await expect(page.locator("h1")).toHaveText("Dashboard");
});
```

E2E tests are slow and flaky. A network hiccup or a slow server can make them fail. But they catch real bugs that unit tests miss.

## The Testing Pyramid

```
        /  E2E  \          Few, slow, expensive
       / Integr.  \        Some, moderate speed
      /   Unit      \      Many, fast, cheap
```

Most of your tests should be unit tests. They are fast and reliable. Add integration tests for important user flows. Use E2E tests sparingly for critical paths like checkout or authentication.

## Testing Philosophy in React

The React community has a strong opinion: **test behavior, not implementation.** This means:

- Test what the user sees, not how the component works internally
- Test interactions, not state values
- Test outcomes, not method calls

```jsx
// Do not test: "setState was called with true"
// Do test: "the toggle button shows as active after clicking"
```

This philosophy comes from React Testing Library, which we will cover in detail. The idea is that your tests should resemble how users interact with your app. Users click buttons and read text. They do not inspect component state.

When your tests focus on behavior, you can refactor your implementation freely. Change from useState to useReducer, swap class components for function components, restructure your state. As long as the behavior stays the same, your tests pass. That is the real power of testing the right way.
