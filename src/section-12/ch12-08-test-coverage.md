# Test Coverage

Test coverage tells you what percentage of your code is exercised by tests. It is a useful metric, but I learned the hard way that chasing 100% coverage can be a trap. Let me explain how to use coverage wisely.

## Generating Coverage Reports

Jest has built-in coverage support:

```bash
# Run tests with coverage
npm test -- --coverage
```

This generates a report showing coverage by file:

```
File           | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
utils.ts       |     100 |      100 |     100 |     100
useCounter.ts  |      95 |       80 |     100 |      95
UserCard.tsx   |      85 |       70 |      90 |      85
Dashboard.tsx  |      60 |       40 |      50 |      60
```

The four metrics are:

- **Statements** - Percentage of code statements executed
- **Branches** - Percentage of if/else branches taken
- **Functions** - Percentage of functions called
- **Lines** - Percentage of executable lines run

## Coverage Configuration

Set coverage thresholds to prevent coverage from dropping:

```js
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  collectCoverageFrom: [
    "src/**/*.{ts,tsx}",
    "!src/**/*.d.ts",
    "!src/main.tsx",
    "!src/vite-env.d.ts",
  ],
};
```

If coverage drops below these thresholds, the test run fails. This prevents coverage from slowly eroding as new code is added without tests.

## What to Cover

Focus your testing effort on code where bugs hurt the most:

**High priority:**
- Business logic and data transformations
- User-facing components and interactions
- API calls and error handling
- Authentication and authorization

**Medium priority:**
- Form validation
- Custom hooks
- Utility functions

**Low priority:**
- Simple display components with no logic
- Type definitions
- Third-party wrapper components
- Constants and configuration

```jsx
// Worth testing: business logic
function calculateDiscount(price, tier) {
  const rates = { basic: 0.05, premium: 0.1, vip: 0.2 };
  return price * (rates[tier] || 0);
}

// Probably not worth testing: simple display
function Label({ text }) {
  return <span>{text}</span>;
}
```

## The 100% Coverage Trap

I once worked on a project that required 100% coverage. We spent more time writing tests for getters, setters, and trivial code than for actual business logic. The coverage number looked great, but the app still had bugs in untested user flows.

High coverage does not mean high quality. A test that calls a function without meaningful assertions still counts as covered:

```jsx
// This counts as "covered" but tests nothing useful
test("render does not throw", () => {
  render(<ComplexComponent />);
  // No assertions!
});
```

## Practical Coverage Targets

Here is what I aim for:

- **80% overall** is a solid target for most projects
- **90%+ for critical paths** like payment, auth, and data integrity
- **No hard requirement for simple display components**

Coverage is a guide, not a goal. Use it to find gaps in your testing, not as a score to maximize. The best test is one that catches a real bug, not one that ticks a coverage box.
