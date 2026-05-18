# Styled Components

Styled Components is a CSS-in-JS library that lets you write actual CSS inside your JavaScript. It generates unique class names at runtime and supports dynamic styling through props. I used it heavily before switching to Tailwind, and it still has its place.

## Installing

```bash
npm install styled-components
```

## Basic Usage

```jsx
import styled from "styled-components";

const Button = styled.button`
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  background-color: #16a34a;
  color: white;

  &:hover {
    background-color: #15803d;
  }
`;

function App() {
  return <Button>Click Me</Button>;
}
```

You write real CSS syntax inside template literals. Styled Components generates a unique class name and injects the styles into the document head.

## Props for Dynamic Styles

This is where Styled Components really shines. You can pass props to change styles dynamically:

```jsx
const Button = styled.button`
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  background-color: ${(props) =>
    props.$variant === "primary" ? "#16a34a" : "#e5e7eb"};
  color: ${(props) =>
    props.$variant === "primary" ? "white" : "#374151"};

  &:hover {
    background-color: ${(props) =>
      props.$variant === "primary" ? "#15803d" : "#d1d5db"};
  }
`;

// Usage
<Button $variant="primary">Save</Button>
<Button $variant="secondary">Cancel</Button>
```

The `$` prefix marks the prop as transient, meaning it will not be passed to the DOM element. This prevents React warnings about unknown HTML attributes.

## Theming

Styled Components has built-in theme support via Context:

```jsx
import { ThemeProvider } from "styled-components";

const theme = {
  colors: {
    primary: "#16a34a",
    secondary: "#e5e7eb",
    text: "#1f2937",
    background: "#ffffff",
  },
  spacing: {
    sm: "8px",
    md: "16px",
    lg: "24px",
  },
};

function App() {
  return (
    <ThemeProvider theme={theme}>
      <Content />
    </ThemeProvider>
  );
}

const Card = styled.div`
  padding: ${(props) => props.theme.spacing.md};
  background-color: ${(props) => props.theme.colors.background};
  color: ${(props) => props.theme.colors.text};
  border-radius: 8px;
`;
```

## Global Styles

You can define global styles that apply to the entire app:

```jsx
import { createGlobalStyle } from "styled-components";

const GlobalStyle = createGlobalStyle`
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
    background-color: #f9fafb;
  }
`;

function App() {
  return (
    <>
      <GlobalStyle />
      <Content />
    </>
  );
}
```

## Animations

Styled Components includes a `keyframes` helper for animations:

```jsx
import styled, { keyframes } from "styled-components";

const fadeIn = keyframes`
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
`;

const AnimatedCard = styled.div`
  animation: ${fadeIn} 0.3s ease-out;
  padding: 16px;
  border-radius: 8px;
  background: white;
`;
```

Styled Components is powerful and expressive. The main trade-off is runtime overhead from generating styles in JavaScript. For most apps it is negligible, but if performance is critical, Tailwind or CSS Modules may be better choices.
