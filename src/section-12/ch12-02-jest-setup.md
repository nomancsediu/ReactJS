# Jest Setup

Jest is the most popular JavaScript testing framework. It works out of the box with most React projects, but let me walk you through the setup so you understand what each piece does.

## Installation

If you used Vite or Create React App, Jest might already be configured. If not:

```bash
npm install --save-dev jest @types/jest ts-jest
npm install --save-dev @testing-library/react @testing-library/jest-dom
npm install --save-dev jest-environment-jsdom
```

## Configuration

Create a Jest config file:

```js
// jest.config.js
module.exports = {
  testEnvironment: "jsdom",
  transform: {
    "^.+\\.tsx?$": "ts-jest",
  },
  moduleNameMapper: {
    "^@/(.*)$": "<rootDir>/src/$1",
  },
  setupFilesAfterSetup: ["<rootDir>/jest.setup.ts"],
  testPathIgnorePatterns: ["/node_modules/", "/dist/"],
};
```

The `jsdom` environment simulates a browser so your React components can render. The `moduleNameMapper` makes `@/` imports work the same way they do in your app.

## Setup File

The setup file runs before every test suite. Use it to add custom matchers:

```ts
// jest.setup.ts
import "@testing-library/jest-dom";
```

This adds matchers like `toBeInTheDocument()`, `toHaveTextContent()`, and `toBeVisible()`.

## Test Scripts

Add these to your `package.json`:

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

`--watch` reruns tests when files change. `--coverage` generates a coverage report.

## The Basic Test Structure

Jest uses `describe` and `it` (or `test`) to organize tests:

```jsx
// __tests__/formatCurrency.test.ts
import { formatCurrency } from "../utils/formatCurrency";

describe("formatCurrency", () => {
  it("formats whole numbers", () => {
    expect(formatCurrency(10)).toBe("$10.00");
  });

  it("formats decimal numbers", () => {
    expect(formatCurrency(10.5)).toBe("$10.50");
  });

  it("formats zero", () => {
    expect(formatCurrency(0)).toBe("$0.00");
  });

  it("handles negative numbers", () => {
    expect(formatCurrency(-5)).toBe("-$5.00");
  });
});
```

`describe` groups related tests. `it` (or `test`) defines a single test case. `expect` makes an assertion.

## Common Matchers

```jsx
// Equality
expect(value).toBe(5);           // Strict equality (===)
expect(value).toEqual({ a: 1 }); // Deep equality

// Truthiness
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();

// Numbers
expect(value).toBeGreaterThan(3);
expect(value).toBeLessThan(10);

// Strings
expect(value).toMatch(/pattern/);

// Arrays
expect(array).toContain("item");

// Errors
expect(() => fn()).toThrow();
```

## Running Your First Test

```bash
npm test
```

You should see green checkmarks for passing tests. If a test fails, Jest shows you exactly what went wrong: the expected value, the received value, and the line number.

That is the foundation. With Jest configured and running, we are ready to test React components.
