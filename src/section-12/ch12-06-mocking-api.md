# Mocking APIs

Your components fetch data from APIs. But in tests, you do not want to hit real servers. Real APIs are slow, unreliable, and may return different data each time. You need controlled, predictable responses. That is where mocking comes in.

## jest.mock for Module Mocking

The simplest approach is to mock the entire module:

```jsx
// __tests__/UserList.test.jsx
import { render, screen } from "@testing-library/react";
import UserList from "./UserList";

jest.mock("../api/users", () => ({
  fetchUsers: jest.fn(),
}));

import { fetchUsers } from "../api/users";

test("displays users from API", async () => {
  fetchUsers.mockResolvedValue([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);

  render(<UserList />);

  expect(await screen.findByText("Alice")).toBeInTheDocument();
  expect(screen.getByText("Bob")).toBeInTheDocument();
});
```

`mockResolvedValue` makes the function return a promise that resolves with your test data. The component thinks it is calling the real API.

## Mocking fetch Directly

If your component uses `fetch` directly, mock the global `fetch` function:

```jsx
beforeEach(() => {
  global.fetch = jest.fn();
});

afterEach(() => {
  jest.restoreAllMocks();
});

test("loads and displays posts", async () => {
  fetch.mockResolvedValue({
    ok: true,
    json: async () => [{ id: 1, title: "Hello World" }],
  });

  render(<PostList />);

  expect(await screen.findByText("Hello World")).toBeInTheDocument();
  expect(fetch).toHaveBeenCalledWith("/api/posts");
});
```

This verifies both the rendered output and that `fetch` was called with the right URL.

## Mocking Error Responses

Test the sad paths too:

```jsx
test("shows error when API fails", async () => {
  fetch.mockResolvedValue({
    ok: false,
    status: 500,
    json: async () => ({ message: "Server error" }),
  });

  render(<PostList />);

  expect(await screen.findByText(/something went wrong/i)).toBeInTheDocument();
});

test("shows error when network fails", async () => {
  fetch.mockRejectedValue(new Error("Failed to fetch"));

  render(<PostList />);

  expect(await screen.findByText(/check your connection/i)).toBeInTheDocument();
});
```

## MSW: Mock Service Worker

For more realistic mocking, use Mock Service Worker. It intercepts network requests at the browser level:

```jsx
// setup.ts
import { setupServer } from "msw/node";
import { http, HttpResponse } from "msw";

export const server = setupServer(
  http.get("/api/users", () => {
    return HttpResponse.json([
      { id: 1, name: "Alice" },
      { id: 2, name: "Bob" },
    ]);
  })
);

// In your test setup file
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

Override handlers for specific tests:

```jsx
test("handles server error", async () => {
  server.use(
    http.get("/api/users", () => {
      return HttpResponse.error();
    })
  );

  render(<UserList />);
  expect(await screen.findByText(/error/i)).toBeInTheDocument();
});
```

MSW is more setup than `jest.mock`, but it gives you several advantages. It works with any HTTP library, not just `fetch`. It intercepts at the network level, so your code runs the same way it does in production. And you can share handlers between unit tests and E2E tests.

## Choosing Your Approach

| Approach | Best For |
|----------|----------|
| `jest.mock` | Simple module mocking, quick setup |
| `global.fetch` mock | Direct fetch calls, simple apps |
| MSW | Realistic testing, shared handlers, any HTTP library |

I start with `jest.mock` for simple cases. When the app has many API calls or uses different HTTP libraries, I switch to MSW. The right choice depends on how complex your API layer is.
