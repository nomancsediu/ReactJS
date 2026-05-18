# Hook Testing

Custom hooks are one of React's most powerful patterns. They encapsulate reusable logic. But testing them can feel tricky because hooks only work inside components. Fortunately, React Testing Library provides a tool for exactly this.

## renderHook

`renderHook` lets you test a hook without building a wrapper component:

```jsx
import { renderHook } from "@testing-library/react";
import { useCounter } from "./useCounter";

test("initializes with default value", () => {
  const { result } = renderHook(() => useCounter());

  expect(result.current.count).toBe(0);
});

test("initializes with custom value", () => {
  const { result } = renderHook(() => useCounter(10));

  expect(result.current.count).toBe(10);
});
```

`result.current` holds the current return value of your hook. Every time the hook updates, `result.current` reflects the new state.

## Testing State Changes

Call your hook's functions and check the updated state:

```jsx
test("increments and decrements", () => {
  const { result } = renderHook(() => useCounter());

  // Initial state
  expect(result.current.count).toBe(0);

  // Increment
  act(() => {
    result.current.increment();
  });
  expect(result.current.count).toBe(1);

  // Decrement
  act(() => {
    result.current.decrement();
  });
  expect(result.current.count).toBe(0);
});
```

Wrap state updates in `act()`. This ensures React finishes processing the update before you make your assertion. Without `act`, you might read stale state.

## Testing Effects

Hooks with `useEffect` need careful testing. The effect runs after render, so you may need to wait:

```jsx
test("fetches data on mount", async () => {
  const { result } = renderHook(() => useFetchUser("123"));

  // Initially loading
  expect(result.current.loading).toBe(true);

  // Wait for the fetch to complete
  await waitFor(() => {
    expect(result.current.loading).toBe(false);
  });

  expect(result.current.user.name).toBe("Alice");
});
```

## Testing Cleanup

If your hook returns a cleanup function, verify it runs on unmount:

```jsx
test("cleans up subscription on unmount", () => {
  const unsubscribe = jest.fn();
  jest.spyOn(chatApi, "subscribe").mockReturnValue(unsubscribe);

  const { unmount } = renderHook(() => useChat("room-1"));

  expect(unsubscribe).not.toHaveBeenCalled();

  unmount();

  expect(unsubscribe).toHaveBeenCalledTimes(1);
});
```

## Testing with Props That Change

When your hook depends on props, use `rerender` to test updates:

```jsx
test("refetches when userId changes", async () => {
  const { result, rerender } = renderHook(
    ({ id }) => useFetchUser(id),
    { initialProps: { id: "1" } }
  );

  await waitFor(() => {
    expect(result.current.user.name).toBe("Alice");
  });

  // Change the prop
  rerender({ id: "2" });

  await waitFor(() => {
    expect(result.current.user.name).toBe("Bob");
  });
});
```

## A Complete Hook Test

Here is a full example testing a `useToggle` hook:

```jsx
import { renderHook, act } from "@testing-library/react";
import { useToggle } from "./useToggle";

describe("useToggle", () => {
  it("starts false by default", () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current[0]).toBe(false);
  });

  it("starts with provided initial value", () => {
    const { result } = renderHook(() => useToggle(true));
    expect(result.current[0]).toBe(true);
  });

  it("toggles the value", () => {
    const { result } = renderHook(() => useToggle());
    act(() => { result.current[1](); });
    expect(result.current[0]).toBe(true);
    act(() => { result.current[1](); });
    expect(result.current[0]).toBe(false);
  });
});
```

Hook tests are quick to write and run. They give you confidence that your reusable logic works before you plug it into components. Test your hooks, and your component tests become simpler because you trust the hooks they use.
